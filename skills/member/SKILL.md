---
name: 10x-member
description: Individual member operations including personal task views, daily check-ins, progress tracking, and self-service features. Use for viewing your tasks, creating check-ins, tracking your progress, and managing your work. Available to all authenticated users.
allowed-tools: Read, Write, Bash
user-invocable: true
argument-hint: "[command] [args]"
---

# 10X Notion Team Skills - Member Operations

Manage your personal tasks, check-ins, and track your progress.

## Quick Commands

| Command | Description |
|---------|-------------|
| `my-tasks` | View your assigned tasks |
| `my-raci` | View your RACI responsibilities |
| `check-in` | Create daily check-in |
| `progress` | View your progress dashboard |
| `complete` | Mark task as complete |
| `block` | Mark task as blocked |
| `profile` | View/update your profile |

## My Tasks

### View All My Tasks

```
/10x-member my-tasks
/10x-member my-tasks --status "In Progress"
/10x-member my-tasks --priority "High"
/10x-member my-tasks --due-today
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();
const statusFilter = $ARGUMENTS.match(/--status\s+"([^"]+)"/)?.[1];
const priorityFilter = $ARGUMENTS.match(/--priority\s+"([^"]+)"/)?.[1];
const dueToday = $ARGUMENTS.includes("--due-today");

const filters = [
  { property: "Assigned To", relation: { contains: currentUser.id }}
];

if (statusFilter) {
  filters.push({ property: "Status", select: { equals: statusFilter }});
} else {
  // Exclude completed by default
  filters.push({ property: "Status", select: { does_not_equal: "Done" }});
}

if (priorityFilter) {
  filters.push({ property: "Priority", select: { equals: priorityFilter }});
}

if (dueToday) {
  const today = new Date().toISOString().split('T')[0];
  filters.push({ property: "Date", date: { equals: today }});
}

const tasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: filters.length > 1 ? { and: filters } : filters[0],
  sorts: [
    { property: "Priority", direction: "ascending" },
    { property: "Date", direction: "ascending" }
  ]
});

// Group by status
const grouped = {
  "Critical": [],
  "High": [],
  "Medium": [],
  "Low": []
};

for (const task of tasks.results) {
  const priority = task.properties.Priority?.select?.name || "Medium";
  const status = task.properties.Status?.select?.name || "To Do";
  const dueDate = task.properties.Date?.date?.start;
  const raciRole = task.properties["RACI Role"]?.select?.name;

  grouped[priority].push({
    name: task.properties["Task Name"].title[0].plain_text,
    status,
    dueDate,
    raciRole: raciRole ? raciRole.charAt(0) : null,
    id: task.id
  });
}

const userName = currentUser.properties.Name.title[0].plain_text;
const skillId = currentUser.properties["Skill ID"]?.rich_text?.[0]?.plain_text;

return `
📋 My Tasks - ${userName} (${skillId})

🔴 CRITICAL
${grouped["Critical"].map(t =>
  `  • [${t.status}] ${t.name} ${t.dueDate ? `(Due: ${t.dueDate})` : ""} ${t.raciRole ? `[${t.raciRole}]` : ""}`
).join('\n') || "  None"}

🟠 HIGH
${grouped["High"].map(t =>
  `  • [${t.status}] ${t.name} ${t.dueDate ? `(Due: ${t.dueDate})` : ""} ${t.raciRole ? `[${t.raciRole}]` : ""}`
).join('\n') || "  None"}

🟡 MEDIUM
${grouped["Medium"].map(t =>
  `  • [${t.status}] ${t.name} ${t.dueDate ? `(Due: ${t.dueDate})` : ""} ${t.raciRole ? `[${t.raciRole}]` : ""}`
).join('\n') || "  None"}

🟢 LOW
${grouped["Low"].map(t =>
  `  • [${t.status}] ${t.name} ${t.dueDate ? `(Due: ${t.dueDate})` : ""} ${t.raciRole ? `[${t.raciRole}]` : ""}`
).join('\n') || "  None"}

Total: ${tasks.results.length} active tasks
`;
```

### View Today's Schedule

```
/10x-member today
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();
const today = new Date().toISOString().split('T')[0];

// Get today's tasks
const tasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Assigned To", relation: { contains: currentUser.id }},
      { property: "Date", date: { equals: today }}
    ]
  },
  sorts: [{ property: "Time Block", direction: "ascending" }]
});

// Get today's meetings/events from check-ins
const checkIn = await notion.databases.query({
  database_id: dailyCheckInsDbId,
  filter: { property: "Date", date: { equals: today }},
  page_size: 1
});

// Get RACI items due today
const raciToday = await notion.databases.query({
  database_id: raciAssignmentsDbId,
  filter: {
    and: [
      { property: "User", relation: { contains: currentUser.id }},
      { property: "Due Date", date: { equals: today }},
      { property: "Status", select: { equals: "Active" }}
    ]
  }
});

// Group tasks by time block
const timeBlocks = {
  "Morning": [],
  "Midday": [],
  "Afternoon": [],
  "Evening": [],
  "Anytime": []
};

for (const task of tasks.results) {
  const timeBlock = task.properties["Time Block"]?.select?.name || "Anytime";
  timeBlocks[timeBlock].push({
    name: task.properties["Task Name"].title[0].plain_text,
    status: task.properties.Status?.select?.name,
    priority: task.properties.Priority?.select?.name
  });
}

return `
📅 Today's Schedule - ${new Date().toLocaleDateString('en-US', { weekday: 'long', month: 'long', day: 'numeric' })}

🌅 MORNING
${timeBlocks["Morning"].map(t => `  • [${t.priority}] ${t.name}`).join('\n') || "  No tasks scheduled"}

☀️ MIDDAY
${timeBlocks["Midday"].map(t => `  • [${t.priority}] ${t.name}`).join('\n') || "  No tasks scheduled"}

🌤️ AFTERNOON
${timeBlocks["Afternoon"].map(t => `  • [${t.priority}] ${t.name}`).join('\n') || "  No tasks scheduled"}

🌙 EVENING
${timeBlocks["Evening"].map(t => `  • [${t.priority}] ${t.name}`).join('\n') || "  No tasks scheduled"}

⏰ ANYTIME
${timeBlocks["Anytime"].map(t => `  • [${t.priority}] ${t.name}`).join('\n') || "  No tasks scheduled"}

${raciToday.results.length > 0 ? `
📋 RACI Due Today:
${raciToday.results.map(r =>
  `  • [${r.properties["RACI Type"].select.name.charAt(0)}] ${r.properties.Name.title[0].plain_text}`
).join('\n')}` : ""}

Total: ${tasks.results.length} tasks today
`;
```

## My RACI Responsibilities

### View RACI Assignments

```
/10x-member my-raci
/10x-member my-raci --type R
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();
const typeFilter = $ARGUMENTS.match(/--type\s+(\w)/)?.[1];

const raciMap = {
  "R": "R - Responsible",
  "A": "A - Accountable",
  "C": "C - Consulted",
  "I": "I - Informed"
};

const filters = [
  { property: "User", relation: { contains: currentUser.id }},
  { property: "Status", select: { equals: "Active" }}
];

if (typeFilter && raciMap[typeFilter.toUpperCase()]) {
  filters.push({ property: "RACI Type", select: { equals: raciMap[typeFilter.toUpperCase()] }});
}

const assignments = await notion.databases.query({
  database_id: raciAssignmentsDbId,
  filter: { and: filters },
  sorts: [
    { property: "RACI Type", direction: "ascending" },
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
  const isOverdue = assignment.properties["Due Date"]?.date?.start &&
    new Date(assignment.properties["Due Date"].date.start) < new Date();

  grouped[raciType].push({
    name: assignment.properties.Name.title[0].plain_text,
    dueDate: assignment.properties["Due Date"]?.date?.start,
    priority: assignment.properties.Priority?.select?.name,
    acknowledged: assignment.properties.Acknowledged?.checkbox,
    isOverdue
  });
}

return `
📋 My RACI Responsibilities

🔴 RESPONSIBLE (I do the work) - ${grouped["R - Responsible"].length}
${grouped["R - Responsible"].map(a =>
  `  ${a.isOverdue ? "⚠️" : "•"} ${a.name} ${a.dueDate ? `(Due: ${a.dueDate})` : ""} ${!a.acknowledged ? "[Pending Ack]" : ""}`
).join('\n') || "  None"}

🟠 ACCOUNTABLE (I approve) - ${grouped["A - Accountable"].length}
${grouped["A - Accountable"].map(a =>
  `  ${a.isOverdue ? "⚠️" : "•"} ${a.name} ${a.dueDate ? `(Due: ${a.dueDate})` : ""}`
).join('\n') || "  None"}

🟡 CONSULTED (I provide input) - ${grouped["C - Consulted"].length}
${grouped["C - Consulted"].map(a =>
  `  • ${a.name} ${!a.acknowledged ? "[Needs Input]" : "[Input Provided]"}`
).join('\n') || "  None"}

🟢 INFORMED (I stay updated) - ${grouped["I - Informed"].length}
${grouped["I - Informed"].map(a =>
  `  • ${a.name}`
).join('\n') || "  None"}

Total: ${assignments.results.length} active assignments
`;
```

## Daily Check-In

### Create Morning Check-In

```
/10x-member check-in
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();
const today = new Date().toISOString().split('T')[0];

// Check if already exists
const existing = await notion.databases.query({
  database_id: dailyCheckInsDbId,
  filter: {
    and: [
      { property: "Date", date: { equals: today }},
      { property: "Related Goals", relation: { contains: currentUser.id }}
    ]
  }
});

if (existing.results.length > 0) {
  return "You already have a check-in for today! View it in your Daily Check-ins.";
}

// Get incomplete tasks from yesterday
const yesterday = new Date();
yesterday.setDate(yesterday.getDate() - 1);
const yesterdayStr = yesterday.toISOString().split('T')[0];

const incompleteTasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Assigned To", relation: { contains: currentUser.id }},
      { property: "Date", date: { equals: yesterdayStr }},
      { property: "Status", select: { does_not_equal: "Done" }}
    ]
  }
});

// Get today's tasks
const todayTasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Assigned To", relation: { contains: currentUser.id }},
      { property: "Date", date: { equals: today }}
    ]
  },
  sorts: [{ property: "Time Block", direction: "ascending" }]
});

// Get active goals
const goals = await notion.databases.query({
  database_id: goalsDbId,
  filter: {
    and: [
      { property: "Owner", relation: { contains: currentUser.id }},
      { property: "Status", select: { equals: "In Progress" }}
    ]
  },
  page_size: 5
});

// Get pending RACI items
const raciItems = await notion.databases.query({
  database_id: raciAssignmentsDbId,
  filter: {
    and: [
      { property: "User", relation: { contains: currentUser.id }},
      { property: "Status", select: { equals: "Active" }},
      { property: "Acknowledged", checkbox: { equals: false }}
    ]
  }
});

const userName = currentUser.properties.Name.title[0].plain_text;

// Create check-in page
const checkIn = await notion.pages.create({
  parent: { database_id: dailyCheckInsDbId },
  properties: {
    "Date": { date: { start: today }},
    "Related Goals": goals.results.length > 0 ? { relation: goals.results.map(g => ({ id: g.id })) } : undefined
  },
  children: [
    {
      type: "heading_1",
      heading_1: {
        rich_text: [{ text: { content: `🌅 Daily Check-In - ${userName}` }}],
        color: "blue_background"
      }
    },
    {
      type: "callout",
      callout: {
        rich_text: [{ text: { content: new Date().toLocaleDateString('en-US', { weekday: 'long', month: 'long', day: 'numeric', year: 'numeric' }) }}],
        icon: { emoji: "📅" }
      }
    },
    ...(incompleteTasks.results.length > 0 ? [
      {
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: "⚠️ Carried Over from Yesterday" }}],
          color: "yellow_background"
        }
      },
      ...incompleteTasks.results.map(task => ({
        type: "to_do",
        to_do: {
          rich_text: [{ text: { content: task.properties["Task Name"].title[0].plain_text }}],
          checked: false
        }
      }))
    ] : []),
    {
      type: "heading_2",
      heading_2: { rich_text: [{ text: { content: "📋 Today's Tasks" }}] }
    },
    ...todayTasks.results.map(task => ({
      type: "to_do",
      to_do: {
        rich_text: [{
          text: {
            content: `[${task.properties["Time Block"]?.select?.name || "Anytime"}] ${task.properties["Task Name"].title[0].plain_text}`
          }
        }],
        checked: false
      }
    })),
    ...(raciItems.results.length > 0 ? [
      {
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: "📋 RACI Items Needing Attention" }}],
          color: "orange_background"
        }
      },
      ...raciItems.results.map(raci => ({
        type: "bulleted_list_item",
        bulleted_list_item: {
          rich_text: [{
            text: {
              content: `[${raci.properties["RACI Type"].select.name.charAt(0)}] ${raci.properties.Name.title[0].plain_text}`
            }
          }]
        }
      }))
    ] : []),
    { type: "divider", divider: {} },
    {
      type: "heading_2",
      heading_2: { rich_text: [{ text: { content: "🎯 Top 3 Priorities Today" }}] }
    },
    { type: "numbered_list_item", numbered_list_item: { rich_text: [] }},
    { type: "numbered_list_item", numbered_list_item: { rich_text: [] }},
    { type: "numbered_list_item", numbered_list_item: { rich_text: [] }},
    { type: "divider", divider: {} },
    {
      type: "heading_2",
      heading_2: {
        rich_text: [{ text: { content: "🌙 End of Day Reflection" }}],
        color: "purple_background"
      }
    },
    {
      type: "paragraph",
      paragraph: {
        rich_text: [{ text: { content: "Complete this section at the end of your day:" }}]
      }
    },
    {
      type: "heading_3",
      heading_3: { rich_text: [{ text: { content: "What went well today?" }}] }
    },
    { type: "paragraph", paragraph: { rich_text: [] }},
    {
      type: "heading_3",
      heading_3: { rich_text: [{ text: { content: "What could be improved?" }}] }
    },
    { type: "paragraph", paragraph: { rich_text: [] }},
    {
      type: "heading_3",
      heading_3: { rich_text: [{ text: { content: "What will I focus on tomorrow?" }}] }
    },
    { type: "paragraph", paragraph: { rich_text: [] }}
  ]
});

return `
✅ Daily Check-In Created!

Date: ${today}
Carried Over: ${incompleteTasks.results.length} tasks
Today's Tasks: ${todayTasks.results.length} tasks
RACI Items: ${raciItems.results.length} pending

Remember to:
1. Fill in your top 3 priorities
2. Complete the reflection at end of day
3. Acknowledge any pending RACI items

View: Your Daily Check-ins database
`;
```

## Task Management

### Complete a Task

```
/10x-member complete "API Design"
```

**Implementation:**

```javascript
const taskName = $ARGUMENTS.match(/complete\s+"([^"]+)"/)?.[1] ||
  $ARGUMENTS.replace(/^complete\s+/, "").trim();
const currentUser = await getCurrentUser();

// Find task
const task = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Task Name", title: { contains: taskName }},
      { property: "Assigned To", relation: { contains: currentUser.id }}
    ]
  }
});

if (!task.results.length) {
  return `Task "${taskName}" not found in your assignments`;
}

const taskData = task.results[0];
const requiresApproval = taskData.properties["Requires Approval"]?.checkbox;

if (requiresApproval) {
  // Check for Accountable person
  const raciAccountable = await notion.databases.query({
    database_id: raciAssignmentsDbId,
    filter: {
      and: [
        { property: "Task", relation: { contains: taskData.id }},
        { property: "RACI Type", select: { equals: "A - Accountable" }}
      ]
    }
  });

  if (raciAccountable.results.length) {
    await notion.pages.update({
      page_id: taskData.id,
      properties: {
        "Status": { select: { name: "In Progress" }}
      }
    });

    return `
Task "${taskName}" requires approval.

Status changed to: In Progress (Pending Approval)
Accountable person has been notified for review.

Use /10x-delegate approve "${taskName}" once approved.
`;
  }
}

// Complete the task
await notion.pages.update({
  page_id: taskData.id,
  properties: {
    "Status": { select: { name: "Done" }},
    "Actual Duration": taskData.properties["Estimated Duration"]?.number ?
      { number: taskData.properties["Estimated Duration"].number } : undefined
  }
});

// Update RACI assignment if exists
const raciAssignment = await notion.databases.query({
  database_id: raciAssignmentsDbId,
  filter: {
    and: [
      { property: "Task", relation: { contains: taskData.id }},
      { property: "User", relation: { contains: currentUser.id }}
    ]
  }
});

if (raciAssignment.results.length) {
  await notion.pages.update({
    page_id: raciAssignment.results[0].id,
    properties: {
      "Status": { select: { name: "Completed" }},
      "Completion Date": { date: { start: new Date().toISOString().split('T')[0] }}
    }
  });
}

// Log audit event
await logAuditEvent("Complete", "Task", taskData.id, taskName, { status: "In Progress" }, { status: "Done" });

return `
✅ Task Completed!

Task: ${taskName}
Completed: ${new Date().toLocaleString()}

Great work! Your task list has been updated.
`;
```

### Block a Task

```
/10x-member block "Database Migration" --reason "Waiting for credentials"
```

### Update Task Progress

```
/10x-member update "API Design" --status "In Progress"
/10x-member update "API Design" --progress 50
```

## Progress Dashboard

### View My Progress

```
/10x-member progress
/10x-member progress --week
/10x-member progress --month
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();
const period = $ARGUMENTS.includes("--month") ? 30 : $ARGUMENTS.includes("--week") ? 7 : 7;

const since = new Date();
since.setDate(since.getDate() - period);
const sinceStr = since.toISOString().split('T')[0];

// Get completed tasks
const completedTasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Assigned To", relation: { contains: currentUser.id }},
      { property: "Status", select: { equals: "Done" }},
      { property: "Date", date: { on_or_after: sinceStr }}
    ]
  }
});

// Get all tasks in period
const allTasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Assigned To", relation: { contains: currentUser.id }},
      { property: "Date", date: { on_or_after: sinceStr }}
    ]
  }
});

// Get goals progress
const goals = await notion.databases.query({
  database_id: goalsDbId,
  filter: {
    and: [
      { property: "Owner", relation: { contains: currentUser.id }},
      { property: "Status", select: { does_not_equal: "Completed" }},
      { property: "Status", select: { does_not_equal: "Abandoned" }}
    ]
  }
});

// Calculate metrics
const completionRate = allTasks.results.length > 0 ?
  Math.round((completedTasks.results.length / allTasks.results.length) * 100) : 0;

const avgGoalProgress = goals.results.length > 0 ?
  Math.round(goals.results.reduce((sum, g) => sum + (g.properties.Progress?.number || 0), 0) / goals.results.length) : 0;

// On-time vs late
let onTime = 0, late = 0;
for (const task of completedTasks.results) {
  const dueDate = task.properties.Date?.date?.start;
  const completedDate = task.last_edited_time.split('T')[0];
  if (dueDate && completedDate > dueDate) {
    late++;
  } else {
    onTime++;
  }
}

const userName = currentUser.properties.Name.title[0].plain_text;
const skillId = currentUser.properties["Skill ID"]?.rich_text?.[0]?.plain_text;

return `
📊 Progress Dashboard - ${userName} (${skillId})
Period: Last ${period} days

📈 TASK COMPLETION
Completed: ${completedTasks.results.length}/${allTasks.results.length} tasks (${completionRate}%)
${"▰".repeat(Math.floor(completionRate/10))}${"▱".repeat(10-Math.floor(completionRate/10))}

On-time: ${onTime} | Late: ${late}
${late === 0 ? "🎉 Perfect timing!" : late <= 2 ? "👍 Good job!" : "⚠️ Consider time management"}

🎯 GOALS PROGRESS
Active Goals: ${goals.results.length}
Average Progress: ${avgGoalProgress}%
${"▰".repeat(Math.floor(avgGoalProgress/10))}${"▱".repeat(10-Math.floor(avgGoalProgress/10))}

${goals.results.slice(0, 5).map(g =>
  `  • ${g.properties.Name.title[0].plain_text}: ${g.properties.Progress?.number || 0}%`
).join('\n')}

📋 PRODUCTIVITY SCORE: ${Math.round((completionRate * 0.6) + (avgGoalProgress * 0.4))}/100
`;
```

## Profile Management

### View My Profile

```
/10x-member profile
```

### Update Profile

```
/10x-member profile --update --timezone "UTC+5:30" --skills "JavaScript, Python"
```

## Acknowledge RACI Assignment

```
/10x-member acknowledge "Project Alpha - Design Review"
```

**Implementation:**

```javascript
const assignmentName = $ARGUMENTS.match(/acknowledge\s+"([^"]+)"/)?.[1];
const currentUser = await getCurrentUser();

const assignment = await notion.databases.query({
  database_id: raciAssignmentsDbId,
  filter: {
    and: [
      { property: "Name", title: { contains: assignmentName }},
      { property: "User", relation: { contains: currentUser.id }}
    ]
  }
});

if (!assignment.results.length) {
  return `Assignment "${assignmentName}" not found`;
}

await notion.pages.update({
  page_id: assignment.results[0].id,
  properties: {
    "Acknowledged": { checkbox: true },
    "Acknowledged Date": { date: { start: new Date().toISOString().split('T')[0] }}
  }
});

return `
✅ Assignment Acknowledged!

Assignment: ${assignmentName}
RACI Role: ${assignment.results[0].properties["RACI Type"].select.name}
Acknowledged: ${new Date().toLocaleString()}
`;
```

## Troubleshooting

**"Task not found in your assignments"**
→ Check the exact task name
→ Verify you're assigned to the task

**"You need to complete your daily check-in"**
→ Run `/10x-member check-in` to create today's check-in

**"Profile update failed"**
→ Some fields may require Admin permissions to update
→ Contact your Admin for assistance

## Version

Skill Version: 2.0.0
Required Role: Member (all commands)
Compatible with: Notion MCP Server 1.0+
