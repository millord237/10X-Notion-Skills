---
name: 10x-admin
description: Administrative commands for managing users, teams, departments, and roles in 10X Notion Team Skills. Use for adding users, creating teams, managing permissions, and configuring organization settings. Requires Admin or higher access.
allowed-tools: Read, Write, Bash
user-invocable: true
argument-hint: "[command] [args]"
---

# 10X Notion Team Skills - Administration

Manage users, teams, departments, and roles in your organization.

## Available Commands

| Command | Description | Role Required |
|---------|-------------|---------------|
| `add-user` | Add new team member | Admin |
| `remove-user` | Deactivate user | Admin |
| `update-user` | Update user details | Admin |
| `add-team` | Create new team | Admin |
| `add-department` | Create new department | Super Admin |
| `assign-role` | Assign role to user | Admin |
| `list-users` | List all users | Team Lead |
| `list-teams` | List all teams | Team Lead |
| `list-roles` | List all roles | Admin |
| `audit-log` | View audit trail | Admin |

## User Management

### Add User

```
/10x-admin add-user "John Doe" john@company.com --team "Backend Team" --role "Member"
```

**Implementation:**

```javascript
// Parse arguments
const [name, email] = $ARGUMENTS.match(/add-user\s+"([^"]+)"\s+(\S+)/);
const teamMatch = $ARGUMENTS.match(/--team\s+"([^"]+)"/);
const roleMatch = $ARGUMENTS.match(/--role\s+"([^"]+)"/);

// Get organization and team details
const org = await notion.databases.query({
  database_id: organizationsDbId,
  page_size: 1
});

const team = teamMatch ? await notion.databases.query({
  database_id: teamsDbId,
  filter: { property: "Name", title: { equals: teamMatch[1] }}
}) : null;

const role = await notion.databases.query({
  database_id: rolesDbId,
  filter: { property: "Name", title: { equals: roleMatch?.[1] || "Member" }}
});

// Get department from team
const dept = team?.results[0] ? await notion.pages.retrieve({
  page_id: team.results[0].properties.Department.relation[0]?.id
}) : null;

// Generate Skill ID
const orgCode = org.results[0].properties.Name.title[0].plain_text.substring(0, 3).toUpperCase();
const deptCode = dept ? dept.properties.Code.rich_text[0]?.plain_text : "GEN";
const roleCode = role.results[0].properties.Code.rich_text[0].plain_text;

// Get next sequence number
const existingUsers = await notion.databases.query({
  database_id: skillIdRegistryDbId,
  filter: {
    property: "Skill ID",
    title: { starts_with: `${orgCode}-${deptCode}-${roleCode}-` }
  }
});
const sequence = existingUsers.results.length + 1;
const skillId = `${orgCode}-${deptCode}-${roleCode}-${String(sequence).padStart(3, '0')}`;

// Create user
const user = await notion.pages.create({
  parent: { database_id: usersDbId },
  properties: {
    "Name": { title: [{ text: { content: name }}] },
    "Email": { email: email },
    "Skill ID": { rich_text: [{ text: { content: skillId }}] },
    "Status": { select: { name: "Active" }},
    "Employment Type": { select: { name: "Full-Time" }},
    "Start Date": { date: { start: new Date().toISOString().split('T')[0] }},
    "Role": { relation: [{ id: role.results[0].id }] },
    "Organization": { relation: [{ id: org.results[0].id }] },
    "Primary Team": team ? { relation: [{ id: team.results[0].id }] } : undefined,
    "Department": dept ? { relation: [{ id: dept.id }] } : undefined
  }
});

// Register Skill ID
await notion.pages.create({
  parent: { database_id: skillIdRegistryDbId },
  properties: {
    "Skill ID": { title: [{ text: { content: skillId }}] },
    "Status": { select: { name: "Active" }},
    "Created Date": { date: { start: new Date().toISOString().split('T')[0] }},
    "User": { relation: [{ id: user.id }] },
    "Organization": { relation: [{ id: org.results[0].id }] },
    "Rate Limit": { number: 1000 },
    "Usage Count": { number: 0 }
  }
});

// Log action
await logAuditEvent("Create", "User", user.id, name, null, { skillId, team: teamMatch?.[1] });

return `
✅ User Created Successfully!

Name: ${name}
Email: ${email}
Skill ID: ${skillId}
Role: ${roleMatch?.[1] || "Member"}
Team: ${teamMatch?.[1] || "Not assigned"}

The user can now use their Skill ID to access the system.
`;
```

### Update User

```
/10x-admin update-user "John Doe" --role "Team Lead" --team "Frontend Team"
```

### Remove User

```
/10x-admin remove-user "John Doe" --reason "Left company"
```

**Implementation:**

```javascript
// Find user
const user = await notion.databases.query({
  database_id: usersDbId,
  filter: { property: "Name", title: { equals: userName }}
});

if (!user.results.length) {
  return "User not found";
}

// Update status to Offboarded
await notion.pages.update({
  page_id: user.results[0].id,
  properties: {
    "Status": { select: { name: "Offboarded" }},
    "End Date": { date: { start: new Date().toISOString().split('T')[0] }}
  }
});

// Revoke Skill ID
const skillId = user.results[0].properties["Skill ID"].rich_text[0]?.plain_text;
if (skillId) {
  const registry = await notion.databases.query({
    database_id: skillIdRegistryDbId,
    filter: { property: "Skill ID", title: { equals: skillId }}
  });

  if (registry.results.length) {
    await notion.pages.update({
      page_id: registry.results[0].id,
      properties: {
        "Status": { select: { name: "Revoked" }}
      }
    });
  }
}

// Log action
await logAuditEvent("Delete", "User", user.results[0].id, userName, { status: "Active" }, { status: "Offboarded", reason });

return `User ${userName} has been offboarded. Skill ID ${skillId} has been revoked.`;
```

## Team Management

### Add Team

```
/10x-admin add-team "Backend Team" --department "Engineering" --lead "Jane Smith"
```

**Implementation:**

```javascript
// Get department
const dept = await notion.databases.query({
  database_id: departmentsDbId,
  filter: { property: "Name", title: { equals: departmentName }}
});

// Get team lead
const lead = leadName ? await notion.databases.query({
  database_id: usersDbId,
  filter: { property: "Name", title: { equals: leadName }}
}) : null;

// Generate team code
const deptCode = dept.results[0].properties.Code.rich_text[0].plain_text;
const existingTeams = await notion.databases.query({
  database_id: teamsDbId,
  filter: { property: "Department", relation: { contains: dept.results[0].id }}
});
const teamLetter = String.fromCharCode(65 + existingTeams.results.length); // A, B, C...
const teamCode = `${deptCode}-${teamLetter}`;

// Create team
const team = await notion.pages.create({
  parent: { database_id: teamsDbId },
  properties: {
    "Name": { title: [{ text: { content: teamName }}] },
    "Code": { rich_text: [{ text: { content: teamCode }}] },
    "Status": { select: { name: "Active" }},
    "Team Type": { select: { name: "Permanent" }},
    "Department": { relation: [{ id: dept.results[0].id }] },
    "Team Lead": lead?.results[0] ? { relation: [{ id: lead.results[0].id }] } : undefined,
    "Sprint Length": { select: { name: "2 Weeks" }},
    "Created Date": { date: { start: new Date().toISOString().split('T')[0] }}
  }
});

// Log action
await logAuditEvent("Create", "Team", team.id, teamName, null, { department: departmentName, lead: leadName });

return `
✅ Team Created Successfully!

Name: ${teamName}
Code: ${teamCode}
Department: ${departmentName}
Team Lead: ${leadName || "Not assigned"}
`;
```

### Add Department

```
/10x-admin add-department "Engineering" --head "CTO Name" --budget 500000
```

**Implementation:**

```javascript
const org = await notion.databases.query({
  database_id: organizationsDbId,
  page_size: 1
});

// Generate department code
const deptCode = departmentName.substring(0, 3).toUpperCase();

// Get department head
const head = headName ? await notion.databases.query({
  database_id: usersDbId,
  filter: { property: "Name", title: { equals: headName }}
}) : null;

const dept = await notion.pages.create({
  parent: { database_id: departmentsDbId },
  properties: {
    "Name": { title: [{ text: { content: departmentName }}] },
    "Code": { rich_text: [{ text: { content: deptCode }}] },
    "Status": { select: { name: "Active" }},
    "Organization": { relation: [{ id: org.results[0].id }] },
    "Department Head": head?.results[0] ? { relation: [{ id: head.results[0].id }] } : undefined,
    "Budget": budget ? { number: budget } : undefined,
    "Created Date": { date: { start: new Date().toISOString().split('T')[0] }}
  }
});

// Log action
await logAuditEvent("Create", "Department", dept.id, departmentName, null, { head: headName, budget });

return `
✅ Department Created Successfully!

Name: ${departmentName}
Code: ${deptCode}
Department Head: ${headName || "Not assigned"}
Budget: ${budget ? `$${budget.toLocaleString()}` : "Not set"}
`;
```

## Role Management

### Assign Role

```
/10x-admin assign-role "John Doe" --role "Team Lead"
```

### List Roles with Permissions

```
/10x-admin list-roles
```

**Output:**

```
┌─────────────────┬─────────┬───────────┬───────────┬──────────┬─────────┐
│ Role            │ Delegate│ Manage    │ View All  │ Reports  │ Scope   │
│                 │         │ Users     │ Teams     │          │         │
├─────────────────┼─────────┼───────────┼───────────┼──────────┼─────────┤
│ Super Admin     │   ✅    │    ✅     │    ✅     │    ✅    │ Org     │
│ Admin           │   ✅    │    ✅     │    ✅     │    ✅    │ Org     │
│ Department Head │   ✅    │    ✅     │    ❌     │    ✅    │ Dept    │
│ Team Lead       │   ✅    │    ❌     │    ❌     │    ✅    │ Team    │
│ Senior Member   │   ✅    │    ❌     │    ❌     │    ✅    │ Team    │
│ Member          │   ❌    │    ❌     │    ❌     │    ❌    │ Self    │
│ Viewer          │   ❌    │    ❌     │    ❌     │    ❌    │ Self    │
└─────────────────┴─────────┴───────────┴───────────┴──────────┴─────────┘
```

## Listing Commands

### List Users

```
/10x-admin list-users
/10x-admin list-users --team "Backend Team"
/10x-admin list-users --department "Engineering"
/10x-admin list-users --role "Team Lead"
```

### List Teams

```
/10x-admin list-teams
/10x-admin list-teams --department "Engineering"
```

## Audit Log

### View Recent Activity

```
/10x-admin audit-log
/10x-admin audit-log --user "John Doe"
/10x-admin audit-log --action "Delete"
/10x-admin audit-log --days 7
```

**Implementation:**

```javascript
const filters = [];

if (userName) {
  const user = await notion.databases.query({
    database_id: usersDbId,
    filter: { property: "Name", title: { equals: userName }}
  });
  if (user.results.length) {
    filters.push({ property: "User", relation: { contains: user.results[0].id }});
  }
}

if (action) {
  filters.push({ property: "Action", select: { equals: action }});
}

if (days) {
  const since = new Date();
  since.setDate(since.getDate() - days);
  filters.push({ property: "Timestamp", date: { on_or_after: since.toISOString() }});
}

const logs = await notion.databases.query({
  database_id: auditLogDbId,
  filter: filters.length > 1 ? { and: filters } : filters[0],
  sorts: [{ property: "Timestamp", direction: "descending" }],
  page_size: 50
});

return logs.results.map(log => ({
  timestamp: log.properties.Timestamp.date.start,
  action: log.properties.Action.select.name,
  entity: log.properties["Entity Type"].select.name,
  name: log.properties["Entity Name"].title[0].plain_text,
  success: log.properties.Success.checkbox
}));
```

## Bulk Operations

### Import Users from CSV

```
/10x-admin import-users --file "users.csv"
```

CSV Format:
```csv
name,email,department,team,role
John Doe,john@company.com,Engineering,Backend Team,Member
Jane Smith,jane@company.com,Marketing,Growth Team,Team Lead
```

### Export Organization Data

```
/10x-admin export --format json
/10x-admin export --format csv --entity users
```

## Helper Function: Audit Logging

```javascript
async function logAuditEvent(action, entityType, entityId, entityName, oldValue, newValue) {
  // Get current user from context
  const currentUser = await getCurrentUser();
  const org = await getCurrentOrganization();

  await notion.pages.create({
    parent: { database_id: auditLogDbId },
    properties: {
      "Entity Name": { title: [{ text: { content: entityName }}] },
      "Timestamp": { date: { start: new Date().toISOString() }},
      "Action": { select: { name: action }},
      "Entity Type": { select: { name: entityType }},
      "Entity ID": { rich_text: [{ text: { content: entityId }}] },
      "User": currentUser ? { relation: [{ id: currentUser.id }] } : undefined,
      "Organization": org ? { relation: [{ id: org.id }] } : undefined,
      "Old Value": oldValue ? { rich_text: [{ text: { content: JSON.stringify(oldValue) }}] } : undefined,
      "New Value": newValue ? { rich_text: [{ text: { content: JSON.stringify(newValue) }}] } : undefined,
      "Success": { checkbox: true }
    }
  });
}
```

## Permission Checks

Before executing any admin command, verify permissions:

```javascript
async function checkPermission(requiredPermission, userId) {
  const user = await notion.pages.retrieve({ page_id: userId });
  const roleId = user.properties.Role.relation[0]?.id;

  if (!roleId) return false;

  const role = await notion.pages.retrieve({ page_id: roleId });

  const permissionMap = {
    "manage_users": "Can Manage Users",
    "manage_teams": "Can Manage Teams",
    "delegate": "Can Delegate",
    "configure_system": "Can Configure System",
    "delete_items": "Can Delete Items",
    "access_reports": "Can Access Reports"
  };

  const propertyName = permissionMap[requiredPermission];
  return role.properties[propertyName]?.checkbox || false;
}

// Usage
if (!await checkPermission("manage_users", currentUserId)) {
  return "Permission denied: You don't have access to manage users";
}
```

## Troubleshooting

**"Permission denied"**
→ Your role doesn't have the required permission
→ Contact an Admin to upgrade your role

**"User not found"**
→ Check the exact name spelling
→ Use `/10x-admin list-users` to see all users

**"Team not found"**
→ Team may not exist or is in a different department
→ Use `/10x-admin list-teams` to see all teams

## Version

Skill Version: 2.0.0
Required Role: Admin (varies by command)
Compatible with: Notion MCP Server 1.0+
