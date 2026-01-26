---
name: 10x-setup
description: Initialize organization with complete team management system including RACI matrix, roles, departments, teams, and users. Use when user wants to setup their organization, create team structure, or initialize the 10X Notion Team Skills system. Requires Super Admin access.
allowed-tools: Read, Write, Bash
user-invocable: true
argument-hint: "[organization-name]"
---

# 10X Notion Team Skills - Organization Setup

Initialize your organization with a complete team management system in Notion.

## What This Creates

When you run `/10x-setup [Organization Name]`, I will create:

### 1. Core Management Databases
- **Organizations** - Company/org container with settings
- **Departments** - Business units with budgets and headcounts
- **Teams** - Work groups with capacity planning
- **Users** - Team members with Skill IDs and permissions
- **Roles** - Permission templates with access levels

### 2. Collaboration Databases
- **RACI Assignments** - Responsibility matrix (Responsible, Accountable, Consulted, Informed)
- **Skill ID Registry** - Central ID mapping for user-to-Notion sync
- **Audit Log** - Activity tracking and compliance

### 3. Enhanced Productivity Databases
- **Goals** (enhanced) - With ownership, RACI, team assignment, approval workflow
- **Tasks** (enhanced) - With delegation, visibility, team assignment
- **Projects** (enhanced) - With PM, sponsors, stakeholders, budget tracking

### 4. Default Role Configuration
| Role | Can Delegate | Can Manage Users | Can View All Teams | Scope |
|------|-------------|------------------|-------------------|-------|
| Super Admin | Yes | Yes | Yes | Organization |
| Admin | Yes | Yes | Yes | Organization |
| Department Head | Yes | Yes | No | Department |
| Team Lead | Yes | No | No | Team |
| Senior Member | Yes | No | No | Team |
| Member | No | No | No | Self |
| Viewer | No | No | No | Self |

## Usage

```
/10x-setup "My Company"
→ Creates organization "My Company"
→ Generates all 8 management databases
→ Creates default roles
→ Sets up first Super Admin user (you)
→ Generates your Skill ID

/10x-setup "My Company" --with-sample-data
→ Same as above plus sample departments/teams
```

## Implementation

### Step 1: Create Organization

```javascript
// Create the main organization
const org = await notion.pages.create({
  parent: { database_id: organizationsDbId },
  properties: {
    "Name": { title: [{ text: { content: $ARGUMENTS || "My Organization" }}] },
    "Status": { select: { name: "Active" }},
    "Plan": { select: { name: "Professional" }},
    "Created Date": { date: { start: new Date().toISOString().split('T')[0] }},
    "Max Users": { number: 100 },
    "Max Teams": { number: 50 }
  }
});

const orgCode = ($ARGUMENTS || "ORG").substring(0, 3).toUpperCase();
```

### Step 2: Create Default Roles

```javascript
const defaultRoles = [
  {
    name: "Super Admin",
    code: "SA",
    accessLevel: "Super Admin",
    scope: "Organization",
    canDelegate: true,
    canCreateGoals: true,
    canManageUsers: true,
    canManageTeams: true,
    canAccessReports: true,
    canConfigureSystem: true,
    canDeleteItems: true
  },
  {
    name: "Admin",
    code: "AD",
    accessLevel: "Admin",
    scope: "Organization",
    canDelegate: true,
    canCreateGoals: true,
    canManageUsers: true,
    canManageTeams: true,
    canAccessReports: true,
    canConfigureSystem: false,
    canDeleteItems: true
  },
  {
    name: "Department Head",
    code: "DH",
    accessLevel: "Admin",
    scope: "Department",
    canDelegate: true,
    canCreateGoals: true,
    canManageUsers: true,
    canManageTeams: true,
    canAccessReports: true,
    canConfigureSystem: false,
    canDeleteItems: false
  },
  {
    name: "Team Lead",
    code: "TL",
    accessLevel: "Team Lead",
    scope: "Team",
    canDelegate: true,
    canCreateGoals: true,
    canManageUsers: false,
    canManageTeams: false,
    canAccessReports: true,
    canConfigureSystem: false,
    canDeleteItems: false
  },
  {
    name: "Senior Member",
    code: "SM",
    accessLevel: "Member",
    scope: "Team",
    canDelegate: true,
    canCreateGoals: true,
    canManageUsers: false,
    canManageTeams: false,
    canAccessReports: true,
    canConfigureSystem: false,
    canDeleteItems: false
  },
  {
    name: "Member",
    code: "MB",
    accessLevel: "Member",
    scope: "Self",
    canDelegate: false,
    canCreateGoals: true,
    canManageUsers: false,
    canManageTeams: false,
    canAccessReports: false,
    canConfigureSystem: false,
    canDeleteItems: false
  },
  {
    name: "Viewer",
    code: "VW",
    accessLevel: "Viewer",
    scope: "Self",
    canDelegate: false,
    canCreateGoals: false,
    canManageUsers: false,
    canManageTeams: false,
    canAccessReports: false,
    canConfigureSystem: false,
    canDeleteItems: false
  }
];

const roleIds = {};
for (const role of defaultRoles) {
  const created = await notion.pages.create({
    parent: { database_id: rolesDbId },
    properties: {
      "Name": { title: [{ text: { content: role.name }}] },
      "Code": { rich_text: [{ text: { content: role.code }}] },
      "Access Level": { select: { name: role.accessLevel }},
      "Scope": { select: { name: role.scope }},
      "Status": { select: { name: "Active" }},
      "Can Delegate": { checkbox: role.canDelegate },
      "Can Create Goals": { checkbox: role.canCreateGoals },
      "Can Manage Users": { checkbox: role.canManageUsers },
      "Can Manage Teams": { checkbox: role.canManageTeams },
      "Can Access Reports": { checkbox: role.canAccessReports },
      "Can Configure System": { checkbox: role.canConfigureSystem },
      "Can Delete Items": { checkbox: role.canDeleteItems }
    }
  });
  roleIds[role.code] = created.id;
}
```

### Step 3: Generate Skill ID

```javascript
// Generate Skill ID for first user (Super Admin)
function generateSkillId(orgCode, deptCode, roleCode, sequence) {
  const seq = String(sequence).padStart(3, '0');
  return `${orgCode}-${deptCode}-${roleCode}-${seq}`;
}

// For the initial Super Admin, use ADM department
const adminSkillId = generateSkillId(orgCode, "ADM", "SA", 1);
// Example: "MYC-ADM-SA-001"

// Create Skill ID Registry entry
await notion.pages.create({
  parent: { database_id: skillIdRegistryDbId },
  properties: {
    "Skill ID": { title: [{ text: { content: adminSkillId }}] },
    "Status": { select: { name: "Active" }},
    "Created Date": { date: { start: new Date().toISOString().split('T')[0] }},
    "Rate Limit": { number: 10000 },
    "Usage Count": { number: 0 }
  }
});
```

### Step 4: Create First User (You as Super Admin)

```javascript
const superAdmin = await notion.pages.create({
  parent: { database_id: usersDbId },
  properties: {
    "Name": { title: [{ text: { content: "Admin User" }}] },
    "Skill ID": { rich_text: [{ text: { content: adminSkillId }}] },
    "Status": { select: { name: "Active" }},
    "Employment Type": { select: { name: "Full-Time" }},
    "Start Date": { date: { start: new Date().toISOString().split('T')[0] }},
    "Role": { relation: [{ id: roleIds["SA"] }] },
    "Organization": { relation: [{ id: org.id }] }
  }
});

// Link admin to organization
await notion.pages.update({
  page_id: org.id,
  properties: {
    "Admin Users": { relation: [{ id: superAdmin.id }] },
    "All Users": { relation: [{ id: superAdmin.id }] }
  }
});
```

### Step 5: Create Sample Structure (Optional)

```javascript
// If --with-sample-data flag is provided
if ($ARGUMENTS.includes("--with-sample-data")) {
  // Create sample departments
  const departments = [
    { name: "Engineering", code: "ENG", color: "Blue" },
    { name: "Marketing", code: "MKT", color: "Purple" },
    { name: "Sales", code: "SLS", color: "Green" },
    { name: "Human Resources", code: "HR", color: "Orange" },
    { name: "Finance", code: "FIN", color: "Yellow" }
  ];

  for (const dept of departments) {
    const deptPage = await notion.pages.create({
      parent: { database_id: departmentsDbId },
      properties: {
        "Name": { title: [{ text: { content: dept.name }}] },
        "Code": { rich_text: [{ text: { content: dept.code }}] },
        "Status": { select: { name: "Active" }},
        "Color": { select: { name: dept.color }},
        "Organization": { relation: [{ id: org.id }] },
        "Created Date": { date: { start: new Date().toISOString().split('T')[0] }}
      }
    });

    // Create a sample team for each department
    await notion.pages.create({
      parent: { database_id: teamsDbId },
      properties: {
        "Name": { title: [{ text: { content: `${dept.name} Team Alpha` }}] },
        "Code": { rich_text: [{ text: { content: `${dept.code}-A` }}] },
        "Status": { select: { name: "Active" }},
        "Team Type": { select: { name: "Permanent" }},
        "Department": { relation: [{ id: deptPage.id }] },
        "Sprint Length": { select: { name: "2 Weeks" }},
        "Created Date": { date: { start: new Date().toISOString().split('T')[0] }}
      }
    });
  }
}
```

### Step 6: Log Setup in Audit Log

```javascript
await notion.pages.create({
  parent: { database_id: auditLogDbId },
  properties: {
    "Entity Name": { title: [{ text: { content: `Organization Setup: ${$ARGUMENTS || "My Organization"}` }}] },
    "Timestamp": { date: { start: new Date().toISOString() }},
    "Action": { select: { name: "Create" }},
    "Entity Type": { select: { name: "Setting" }},
    "User": { relation: [{ id: superAdmin.id }] },
    "Organization": { relation: [{ id: org.id }] },
    "Success": { checkbox: true },
    "New Value": { rich_text: [{ text: { content: JSON.stringify({
      organization: org.id,
      roles_created: Object.keys(roleIds).length,
      skill_id: adminSkillId
    })}}]}
  }
});
```

## Output

After setup completes, you'll receive:

```
✅ Organization Setup Complete!

Organization: My Company
Organization Code: MYC

Created:
├── Organizations Database
├── Departments Database
├── Teams Database
├── Users Database
├── Roles Database (7 default roles)
├── RACI Assignments Database
├── Skill ID Registry Database
└── Audit Log Database

Your Details:
├── Role: Super Admin
├── Skill ID: MYC-ADM-SA-001
└── Access: Full system access

Next Steps:
1. Add departments: /10x-admin add-department "Engineering"
2. Create teams: /10x-admin add-team "Backend Team" --department "Engineering"
3. Invite users: /10x-admin add-user "John Doe" john@company.com --team "Backend Team"
4. Start delegating: /10x-delegate "Complete API design" --to "John Doe" --raci R

View your dashboard: [Link to Notion workspace]
```

## Database Relations Diagram

```
┌─────────────────┐
│  Organizations  │
└────────┬────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
┌────────┐  ┌────────┐
│ Roles  │  │ Depts  │
└────────┘  └───┬────┘
                │
           ┌────┴────┐
           │         │
           ▼         ▼
      ┌────────┐  ┌────────┐
      │ Teams  │  │ Users  │◄──┐
      └────┬───┘  └───┬────┘   │
           │          │        │
           └────┬─────┘        │
                │              │
                ▼              │
         ┌──────────────┐      │
         │     RACI     │──────┘
         │ Assignments  │
         └──────────────┘
                │
                ▼
         ┌──────────────┐
         │    Tasks     │
         │    Goals     │
         │   Projects   │
         └──────────────┘
```

## Troubleshooting

**"Permission denied"**
→ You need Super Admin access to run setup
→ Contact existing Super Admin or check Notion integration permissions

**"Organization already exists"**
→ Each organization name must be unique
→ Use `/10x-admin list-orgs` to see existing organizations

**"Database creation failed"**
→ Verify Notion MCP server is configured
→ Check integration has access to workspace
→ Ensure parent page exists

## Version

Skill Version: 2.0.0
Required Role: Super Admin
Compatible with: Notion MCP Server 1.0+
