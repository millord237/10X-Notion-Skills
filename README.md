# 10X Notion Team Skills

**Enterprise Team Management for Notion with RACI Matrix Support**

Transform Notion into a complete team management powerhouse with multi-team collaboration, delegation, RACI matrix, and organization-wide analytics.

[![Version](https://img.shields.io/badge/version-2.0.0-blue.svg)](https://github.com/10xin/10x-notion-team-skills)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-purple.svg)](https://claude.ai)

## Features

### Team Management
- **Organizations, Departments, Teams** - Complete organizational hierarchy
- **User Management** - Onboarding, offboarding, role assignment
- **Skill ID System** - Unique identifiers mapping users to Notion
- **Role-Based Access Control** - Granular permissions per role

### RACI Matrix
- **Responsibility Assignment** - R (Responsible), A (Accountable), C (Consulted), I (Informed)
- **Compliance Tracking** - Ensure every task has proper RACI coverage
- **Approval Workflows** - Accountable users approve completed work
- **Matrix Visualization** - Clear view of responsibilities

### Delegation & Collaboration
- **Task Delegation** - Assign tasks with RACI roles
- **Escalation Workflows** - Handle blocked tasks
- **Cross-Team Visibility** - Based on role permissions
- **Audit Logging** - Track all actions

### Analytics & Reporting
- **Team Performance** - Completion rates, velocity, capacity
- **Department Analytics** - Cross-team comparisons
- **Organization Reports** - Company-wide KPIs
- **RACI Compliance** - Coverage and violation reports

## Quick Start

### 1. Install the Plugin

```bash
# Add to your Claude Code
claude /plugin install 10x-notion-team-skills
```

Or manually add to `.claude/settings.json`:

```json
{
  "plugins": [
    {
      "name": "10x-notion-team-skills",
      "source": "github",
      "repo": "10xin/10x-notion-team-skills"
    }
  ]
}
```

### 2. Configure Notion MCP Server

Add to your Claude Desktop config (`~/.config/claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "OPENAPI_MCP_HEADERS": "{\"Authorization\": \"Bearer YOUR_NOTION_TOKEN\", \"Notion-Version\": \"2022-06-28\"}"
      }
    }
  }
}
```

### 3. Initialize Your Organization

```
/10x-setup "My Company"
```

This creates:
- All management databases (Organizations, Departments, Teams, Users, Roles)
- RACI Assignments database
- Skill ID Registry
- Audit Log
- Default roles with permissions
- Your Super Admin account with Skill ID

## Commands

### Setup & Administration

| Command | Description | Role |
|---------|-------------|------|
| `/10x-setup "Org Name"` | Initialize organization | Super Admin |
| `/10x-admin add-user "Name" email` | Add team member | Admin |
| `/10x-admin add-team "Team"` | Create team | Admin |
| `/10x-admin add-department "Dept"` | Create department | Super Admin |
| `/10x-admin assign-role "User" --role "Role"` | Assign role | Admin |

### Delegation & RACI

| Command | Description | Role |
|---------|-------------|------|
| `/10x-delegate "Task" --to "User" --raci R` | Delegate with RACI | Team Lead |
| `/10x-delegate raci-matrix --project "Project"` | View RACI matrix | Admin |
| `/10x-delegate my-assignments` | View your RACI duties | Member |
| `/10x-delegate approve "Task"` | Approve as Accountable | Accountable |

### Team Operations

| Command | Description | Role |
|---------|-------------|------|
| `/10x-team board` | View team kanban | Member |
| `/10x-team standup` | Run daily standup | Team Lead |
| `/10x-team sprint new --name "Sprint 1"` | Create sprint | Team Lead |
| `/10x-team capacity` | View team capacity | Team Lead |
| `/10x-team velocity` | View velocity trends | Team Lead |

### Personal Operations

| Command | Description | Role |
|---------|-------------|------|
| `/10x-member my-tasks` | View your tasks | Member |
| `/10x-member my-raci` | View RACI assignments | Member |
| `/10x-member check-in` | Create daily check-in | Member |
| `/10x-member complete "Task"` | Mark task complete | Member |
| `/10x-member progress` | View your dashboard | Member |

### Analytics

| Command | Description | Role |
|---------|-------------|------|
| `/10x-analytics team-report` | Team performance | Team Lead |
| `/10x-analytics dept-report` | Department metrics | Dept Head |
| `/10x-analytics org-report` | Organization KPIs | Admin |
| `/10x-analytics raci-compliance` | RACI coverage report | Admin |
| `/10x-analytics kpi` | Key Performance Indicators | Admin |

## Role Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                        SUPER ADMIN                          │
│  Full system access, can configure everything               │
├─────────────────────────────────────────────────────────────┤
│                           ADMIN                             │
│  Manage users/teams across organization                     │
├─────────────────────────────────────────────────────────────┤
│                      DEPARTMENT HEAD                        │
│  Manage users/teams within department                       │
├─────────────────────────────────────────────────────────────┤
│                        TEAM LEAD                            │
│  Delegate tasks, run standups, view reports                 │
├─────────────────────────────────────────────────────────────┤
│                      SENIOR MEMBER                          │
│  Delegate within team, create goals/tasks                   │
├─────────────────────────────────────────────────────────────┤
│                         MEMBER                              │
│  Work on assigned tasks, personal tracking                  │
├─────────────────────────────────────────────────────────────┤
│                         VIEWER                              │
│  Read-only access to assigned items                         │
└─────────────────────────────────────────────────────────────┘
```

## RACI Matrix

The RACI framework ensures clear responsibility assignment:

| Role | Symbol | Description | Actions |
|------|--------|-------------|---------|
| **Responsible** | R | Does the work | Can complete task |
| **Accountable** | A | Final approver (ONE per task) | Approves completion |
| **Consulted** | C | Provides input | Asked before decisions |
| **Informed** | I | Kept updated | Receives status updates |

Example RACI Matrix:

```
┌──────────────────┬─────────┬─────────┬─────────┬─────────┐
│ Task             │ Dev     │ PM      │ Design  │ CEO     │
├──────────────────┼─────────┼─────────┼─────────┼─────────┤
│ Requirements     │    C    │    R    │    C    │    I    │
│ Development      │    R    │    A    │    C    │    I    │
│ Testing          │    R    │    C    │    I    │    I    │
│ Launch           │    C    │    R    │    C    │    A    │
└──────────────────┴─────────┴─────────┴─────────┴─────────┘
```

## Skill ID System

Every user gets a unique Skill ID for tracking:

```
Format: {ORG}-{DEPT}-{ROLE}-{SEQ}

Examples:
  10X-ENG-TL-001  → 10X Engineering Team Lead #1
  10X-MKT-MB-003  → 10X Marketing Member #3
  10X-FIN-AD-001  → 10X Finance Admin #1
```

## Database Architecture

```
Organizations
    ├── Departments
    │       ├── Teams
    │       │      └── Users ←──┐
    │       └── Goals           │
    │                           │
    ├── Roles ──────────────────┤
    │                           │
    └── Audit Log               │
                                │
RACI Assignments ───────────────┤
         │                      │
         ├── Tasks ─────────────┘
         ├── Goals
         └── Projects
```

## File Structure

```
10x-notion-team-skills/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── skills/
│   ├── setup/SKILL.md           # Organization setup
│   ├── admin/SKILL.md           # Administration
│   ├── delegation/SKILL.md      # RACI & delegation
│   ├── team/SKILL.md            # Team operations
│   ├── member/SKILL.md          # Personal operations
│   └── analytics/SKILL.md       # Reporting
├── templates/
│   ├── database-schemas.json    # Personal databases
│   └── team-database-schemas.json # Team databases
├── reference/
│   └── database-schemas.md      # Schema documentation
├── examples/
│   └── complete-workflows.md    # Usage examples
└── README.md                    # This file
```

## Requirements

- **Claude Code** v1.0.0 or higher
- **Notion Integration** with read/write access
- **Notion MCP Server** (@notionhq/notion-mcp-server)

## Pricing Tiers

| Tier | Users | Teams | Features |
|------|-------|-------|----------|
| **Free** | 5 | 2 | Basic task management, team board |
| **Starter** | 25 | 10 | + RACI matrix, delegation, sprints |
| **Professional** | 100 | 50 | + Advanced analytics, audit log |
| **Enterprise** | Unlimited | Unlimited | + SSO, API access, custom integrations |

## Support

- **Documentation**: [Full Docs](reference/)
- **Examples**: [Workflow Examples](examples/)
- **Issues**: [GitHub Issues](https://github.com/10xin/10x-notion-team-skills/issues)

## License

MIT License - see [LICENSE](LICENSE) for details.

## Author

**10X.in** - Building productivity tools for teams

---

Made with ❤️ for teams who want to 10X their productivity
