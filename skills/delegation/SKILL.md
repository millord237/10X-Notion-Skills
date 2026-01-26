---
name: 10x-delegate
description: Delegate tasks and assignments with RACI matrix support. Use for assigning tasks with Responsible, Accountable, Consulted, or Informed roles. Supports task delegation, escalation, and approval workflows. Requires Team Lead or higher for delegation.
allowed-tools: Read, Write, Bash
user-invocable: true
argument-hint: "[task-description] --to [user] --raci [R|A|C|I]"
---

# 10X Notion Team Skills - Delegation & RACI Matrix

Delegate tasks and manage responsibilities using the RACI framework.

## RACI Framework

| Role | Symbol | Description | Can Complete Task | Notified |
|------|--------|-------------|-------------------|----------|
| **Responsible** | R | Does the work | Yes | Yes |
| **Accountable** | A | Final approver (only ONE per task) | Approves only | Yes |
| **Consulted** | C | Provides input before work | No | Yes |
| **Informed** | I | Kept updated on progress | No | Yes |

## Quick Commands

```
/10x-delegate "Design API endpoints" --to "John Doe" --raci R
/10x-delegate "Review API design" --to "Jane Smith" --raci A
/10x-delegate "Provide security input" --to "Security Team" --raci C
/10x-delegate "API progress update" --to "Product Team" --raci I
```

## Delegation Commands

### Basic Delegation

```
/10x-delegate "Complete user authentication" --to "John Doe" --raci R --due "2025-02-15"
```

**Implementation:**

```javascript
// Parse arguments
const taskDescription = $ARGUMENTS.match(/"([^"]+)"/)?.[1];
const toUser = $ARGUMENTS.match(/--to\s+"([^"]+)"/)?.[1];
const raciType = $ARGUMENTS.match(/--raci\s+(\w)/)?.[1];
const dueDate = $ARGUMENTS.match(/--due\s+"([^"]+)"/)?.[1];
const priority = $ARGUMENTS.match(/--priority\s+(\w+)/)?.[1] || "Medium";

// Validate RACI type
const raciMap = {
  "R": "R - Responsible",
  "A": "A - Accountable",
  "C": "C - Consulted",
  "I": "I - Informed"
};

if (!raciMap[raciType.toUpperCase()]) {
  return "Invalid RACI type. Use R, A, C, or I";
}

// Get current user (delegator)
const currentUser = await getCurrentUser();

// Check delegation permission
if (!await checkPermission("delegate", currentUser.id)) {
  return "Permission denied: You don't have delegation rights";
}

// Get target user
const targetUser = await notion.databases.query({
  database_id: usersDbId,
  filter: { property: "Name", title: { equals: toUser }}
});

if (!targetUser.results.length) {
  return `User "${toUser}" not found`;
}

// Check if task exists or create new one
let task;
const existingTask = await notion.databases.query({
  database_id: tasksDbId,
  filter: { property: "Task Name", title: { equals: taskDescription }}
});

if (existingTask.results.length) {
  task = existingTask.results[0];
} else {
  // Create new task
  task = await notion.pages.create({
    parent: { database_id: tasksDbId },
    properties: {
      "Task Name": { title: [{ text: { content: taskDescription }}] },
      "Status": { select: { name: "To Do" }},
      "Priority": { select: { name: priority }},
      "Date": dueDate ? { date: { start: dueDate }} : undefined,
      "Assigned To": { relation: [{ id: targetUser.results[0].id }] },
      "Delegated By": { relation: [{ id: currentUser.id }] },
      "RACI Role": { select: { name: raciMap[raciType.toUpperCase()] }},
      "Delegation Date": { date: { start: new Date().toISOString().split('T')[0] }},
      "Team": currentUser.properties["Primary Team"]?.relation?.[0] ?
        { relation: [{ id: currentUser.properties["Primary Team"].relation[0].id }] } : undefined
    }
  });
}

// Create RACI assignment
const raciAssignment = await notion.pages.create({
  parent: { database_id: raciAssignmentsDbId },
  properties: {
    "Name": { title: [{ text: { content: `${taskDescription} - ${toUser}` }}] },
    "RACI Type": { select: { name: raciMap[raciType.toUpperCase()] }},
    "Status": { select: { name: "Active" }},
    "Assignment Date": { date: { start: new Date().toISOString().split('T')[0] }},
    "Due Date": dueDate ? { date: { start: dueDate }} : undefined,
    "Priority": { select: { name: priority }},
    "User": { relation: [{ id: targetUser.results[0].id }] },
    "Task": { relation: [{ id: task.id }] },
    "Assigned By": { relation: [{ id: currentUser.id }] }
  }
});

// Log audit event
await logAuditEvent("Delegate", "Task", task.id, taskDescription, null, {
  assignedTo: toUser,
  raciType: raciMap[raciType.toUpperCase()],
  dueDate
});

return `
✅ Task Delegated Successfully!

Task: ${taskDescription}
Assigned To: ${toUser}
RACI Role: ${raciMap[raciType.toUpperCase()]}
Due Date: ${dueDate || "Not set"}
Priority: ${priority}

${toUser} will be notified of this assignment.
`;
```

### Multi-RACI Assignment

Assign multiple RACI roles for a single task:

```
/10x-delegate "Launch new product" --multi
  --responsible "Product Team"
  --accountable "VP Product"
  --consulted "Legal Team, Marketing"
  --informed "CEO, Board"
```

**Implementation:**

```javascript
// Parse multi-assignment
const taskDescription = $ARGUMENTS.match(/"([^"]+)"/)?.[1];
const responsible = $ARGUMENTS.match(/--responsible\s+"([^"]+)"/)?.[1];
const accountable = $ARGUMENTS.match(/--accountable\s+"([^"]+)"/)?.[1];
const consulted = $ARGUMENTS.match(/--consulted\s+"([^"]+)"/)?.[1]?.split(",").map(s => s.trim());
const informed = $ARGUMENTS.match(/--informed\s+"([^"]+)"/)?.[1]?.split(",").map(s => s.trim());

// Validate: only ONE accountable
if (!accountable) {
  return "Error: Every task must have exactly ONE Accountable person";
}

// Create task
const task = await notion.pages.create({
  parent: { database_id: tasksDbId },
  properties: {
    "Task Name": { title: [{ text: { content: taskDescription }}] },
    "Status": { select: { name: "To Do" }},
    "Requires Approval": { checkbox: true }
  }
});

const assignments = [];

// Assign Responsible
if (responsible) {
  const users = responsible.split(",").map(s => s.trim());
  for (const userName of users) {
    const user = await findUser(userName);
    if (user) {
      await createRaciAssignment(task.id, user.id, "R - Responsible", taskDescription);
      assignments.push({ user: userName, role: "R" });
    }
  }
}

// Assign Accountable (only one)
const accUser = await findUser(accountable);
if (accUser) {
  await createRaciAssignment(task.id, accUser.id, "A - Accountable", taskDescription);
  assignments.push({ user: accountable, role: "A" });

  // Set as approver on task
  await notion.pages.update({
    page_id: task.id,
    properties: {
      "Approved By": { relation: [{ id: accUser.id }] }
    }
  });
}

// Assign Consulted
if (consulted) {
  for (const userName of consulted) {
    const user = await findUser(userName);
    if (user) {
      await createRaciAssignment(task.id, user.id, "C - Consulted", taskDescription);
      assignments.push({ user: userName, role: "C" });
    }
  }
}

// Assign Informed
if (informed) {
  for (const userName of informed) {
    const user = await findUser(userName);
    if (user) {
      await createRaciAssignment(task.id, user.id, "I - Informed", taskDescription);
      assignments.push({ user: userName, role: "I" });
    }
  }
}

return `
✅ RACI Matrix Created for: ${taskDescription}

${assignments.map(a => `[${a.role}] ${a.user}`).join('\n')}

Total assignments: ${assignments.length}
`;
```

### Escalation

Escalate a blocked or delayed task:

```
/10x-delegate escalate "API Design" --reason "Blocked by dependencies" --to "Manager"
```

**Implementation:**

```javascript
const taskName = $ARGUMENTS.match(/escalate\s+"([^"]+)"/)?.[1];
const reason = $ARGUMENTS.match(/--reason\s+"([^"]+)"/)?.[1];
const escalateTo = $ARGUMENTS.match(/--to\s+"([^"]+)"/)?.[1];

// Find task
const task = await notion.databases.query({
  database_id: tasksDbId,
  filter: { property: "Task Name", title: { equals: taskName }}
});

if (!task.results.length) {
  return `Task "${taskName}" not found`;
}

// Get escalation target
const manager = await findUser(escalateTo);
if (!manager) {
  return `User "${escalateTo}" not found`;
}

// Update task status
await notion.pages.update({
  page_id: task.results[0].id,
  properties: {
    "Status": { select: { name: "Blocked" }},
    "Delegation Notes": { rich_text: [{ text: { content: `ESCALATED: ${reason}` }}] }
  }
});

// Create escalation RACI assignment
await notion.pages.create({
  parent: { database_id: raciAssignmentsDbId },
  properties: {
    "Name": { title: [{ text: { content: `ESCALATION: ${taskName}` }}] },
    "RACI Type": { select: { name: "A - Accountable" }},
    "Status": { select: { name: "Active" }},
    "Priority": { select: { name: "Critical" }},
    "User": { relation: [{ id: manager.id }] },
    "Task": { relation: [{ id: task.results[0].id }] },
    "Notes": { rich_text: [{ text: { content: reason }}] }
  }
});

// Log audit event
await logAuditEvent("Escalate", "Task", task.results[0].id, taskName, null, {
  escalatedTo: escalateTo,
  reason
});

return `
⚠️ Task Escalated!

Task: ${taskName}
Escalated To: ${escalateTo}
Reason: ${reason}

${escalateTo} has been notified and assigned as Accountable.
`;
```

## RACI Matrix View

### View RACI for a Project/Goal

```
/10x-delegate raci-matrix --project "Q1 Product Launch"
```

**Output:**

```
┌──────────────────────────┬───────────┬───────────┬───────────┬───────────┐
│ Task/Deliverable         │ Product   │ Eng Lead  │ Marketing │ CEO       │
├──────────────────────────┼───────────┼───────────┼───────────┼───────────┤
│ Requirements Doc         │     R     │     C     │     C     │     I     │
│ Technical Design         │     C     │     R     │     I     │     I     │
│ Development              │     A     │     R     │     I     │     I     │
│ QA Testing               │     C     │     R     │     I     │     I     │
│ Marketing Materials      │     C     │     I     │     R     │     A     │
│ Launch Announcement      │     A     │     I     │     R     │     I     │
│ Post-Launch Review       │     R     │     R     │     R     │     A     │
└──────────────────────────┴───────────┴───────────┴───────────┴───────────┘

Legend: R=Responsible, A=Accountable, C=Consulted, I=Informed
```

### View My RACI Assignments

```
/10x-delegate my-assignments
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();

const assignments = await notion.databases.query({
  database_id: raciAssignmentsDbId,
  filter: {
    and: [
      { property: "User", relation: { contains: currentUser.id }},
      { property: "Status", select: { equals: "Active" }}
    ]
  },
  sorts: [
    { property: "Priority", direction: "ascending" },
    { property: "Due Date", direction: "ascending" }
  ]
});

// Group by RACI type
const grouped = {
  "R - Responsible": [],
  "A - Accountable": [],
  "C - Consulted": [],
  "I - Informed": []
};

for (const assignment of assignments.results) {
  const raciType = assignment.properties["RACI Type"].select.name;
  grouped[raciType].push({
    name: assignment.properties.Name.title[0].plain_text,
    dueDate: assignment.properties["Due Date"]?.date?.start,
    priority: assignment.properties.Priority.select?.name
  });
}

return `
📋 Your RACI Assignments

🔴 RESPONSIBLE (You do the work):
${grouped["R - Responsible"].map(a => `  • ${a.name} ${a.dueDate ? `(Due: ${a.dueDate})` : ""}`).join('\n') || "  None"}

🟠 ACCOUNTABLE (You approve):
${grouped["A - Accountable"].map(a => `  • ${a.name} ${a.dueDate ? `(Due: ${a.dueDate})` : ""}`).join('\n') || "  None"}

🟡 CONSULTED (Provide input):
${grouped["C - Consulted"].map(a => `  • ${a.name}`).join('\n') || "  None"}

🟢 INFORMED (Stay updated):
${grouped["I - Informed"].map(a => `  • ${a.name}`).join('\n') || "  None"}

Total: ${assignments.results.length} active assignments
`;
```

## Approval Workflow

### Request Approval

When a Responsible user completes work:

```
/10x-delegate complete "API Design" --request-approval
```

**Implementation:**

```javascript
const taskName = $ARGUMENTS.match(/complete\s+"([^"]+)"/)?.[1];
const currentUser = await getCurrentUser();

// Find task
const task = await notion.databases.query({
  database_id: tasksDbId,
  filter: { property: "Task Name", title: { equals: taskName }}
});

// Find the Accountable person
const raciAccountable = await notion.databases.query({
  database_id: raciAssignmentsDbId,
  filter: {
    and: [
      { property: "Task", relation: { contains: task.results[0].id }},
      { property: "RACI Type", select: { equals: "A - Accountable" }}
    ]
  }
});

if (!raciAccountable.results.length) {
  // No accountable - auto-complete
  await notion.pages.update({
    page_id: task.results[0].id,
    properties: {
      "Status": { select: { name: "Done" }}
    }
  });
  return "Task marked as complete (no approval required)";
}

// Update task to pending approval
await notion.pages.update({
  page_id: task.results[0].id,
  properties: {
    "Status": { select: { name: "In Progress" }},
    "Requires Approval": { checkbox: true }
  }
});

// Update RACI assignment
await notion.pages.update({
  page_id: raciAccountable.results[0].id,
  properties: {
    "Notes": { rich_text: [{ text: { content: `Pending approval from ${currentUser.properties.Name.title[0].plain_text}` }}] }
  }
});

const accountableUser = raciAccountable.results[0].properties.User.relation[0];
const userName = await notion.pages.retrieve({ page_id: accountableUser.id });

return `
📝 Approval Requested

Task: ${taskName}
Submitted by: You
Approval pending from: ${userName.properties.Name.title[0].plain_text}

The Accountable person will review and approve.
`;
```

### Approve/Reject

When an Accountable user reviews:

```
/10x-delegate approve "API Design"
/10x-delegate reject "API Design" --reason "Needs more detail"
```

## Permission Rules for Delegation

| Action | Super Admin | Admin | Dept Head | Team Lead | Sr Member | Member |
|--------|-------------|-------|-----------|-----------|-----------|--------|
| Delegate to anyone in org | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Delegate within department | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Delegate within team | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Set Accountable | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |
| Escalate tasks | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Approve tasks | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ |

## Helper Functions

```javascript
async function findUser(nameOrId) {
  // Try by name first
  let user = await notion.databases.query({
    database_id: usersDbId,
    filter: { property: "Name", title: { equals: nameOrId }}
  });

  if (user.results.length) return user.results[0];

  // Try by Skill ID
  user = await notion.databases.query({
    database_id: usersDbId,
    filter: { property: "Skill ID", rich_text: { equals: nameOrId }}
  });

  return user.results[0] || null;
}

async function createRaciAssignment(taskId, userId, raciType, taskDescription) {
  const user = await notion.pages.retrieve({ page_id: userId });
  const userName = user.properties.Name.title[0].plain_text;

  return notion.pages.create({
    parent: { database_id: raciAssignmentsDbId },
    properties: {
      "Name": { title: [{ text: { content: `${taskDescription} - ${userName}` }}] },
      "RACI Type": { select: { name: raciType }},
      "Status": { select: { name: "Active" }},
      "Assignment Date": { date: { start: new Date().toISOString().split('T')[0] }},
      "User": { relation: [{ id: userId }] },
      "Task": { relation: [{ id: taskId }] },
      "Assigned By": { relation: [{ id: (await getCurrentUser()).id }] }
    }
  });
}

async function checkDelegationScope(currentUser, targetUser) {
  const currentRole = await getUserRole(currentUser);
  const scope = currentRole.properties.Scope.select.name;

  switch (scope) {
    case "Organization":
      return true; // Can delegate to anyone
    case "Department":
      return currentUser.properties.Department?.relation?.[0]?.id ===
             targetUser.properties.Department?.relation?.[0]?.id;
    case "Team":
      return currentUser.properties["Primary Team"]?.relation?.[0]?.id ===
             targetUser.properties["Primary Team"]?.relation?.[0]?.id;
    case "Self":
      return false; // Cannot delegate
    default:
      return false;
  }
}
```

## Troubleshooting

**"Cannot delegate to this user"**
→ Your role scope doesn't allow delegation to users outside your team/department
→ Contact an Admin to make cross-team assignments

**"Task already has an Accountable person"**
→ Each task can only have ONE Accountable person
→ Remove existing Accountable assignment first

**"Permission denied for approval"**
→ Only Accountable users can approve tasks
→ Check your RACI assignment

## Version

Skill Version: 2.0.0
Required Role: Team Lead (for delegation), varies for other actions
Compatible with: Notion MCP Server 1.0+
