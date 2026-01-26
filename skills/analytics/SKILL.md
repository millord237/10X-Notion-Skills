---
name: 10x-analytics
description: Generate analytics reports for teams, departments, and the organization. Use for viewing performance metrics, generating reports, tracking KPIs, and analyzing productivity trends. Requires Team Lead for team reports, Admin for org-wide reports.
allowed-tools: Read, Write, Bash
user-invocable: true
argument-hint: "[report-type] [args]"
---

# 10X Notion Team Skills - Analytics & Reporting

Generate comprehensive analytics reports for teams, departments, and the entire organization.

## Quick Commands

| Command | Description | Role Required |
|---------|-------------|---------------|
| `team-report` | Generate team performance report | Team Lead |
| `dept-report` | Generate department report | Dept Head |
| `org-report` | Generate organization report | Admin |
| `raci-compliance` | RACI matrix compliance report | Admin |
| `velocity` | Team velocity trends | Team Lead |
| `capacity` | Capacity utilization report | Team Lead |
| `kpi` | Key Performance Indicators | Admin |

## Team Performance Report

### Generate Team Report

```
/10x-analytics team-report
/10x-analytics team-report --team "Backend Team" --period 30
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();
const teamFilter = $ARGUMENTS.match(/--team\s+"([^"]+)"/)?.[1];
const period = parseInt($ARGUMENTS.match(/--period\s+(\d+)/)?.[1]) || 30;

let teamId;
if (teamFilter) {
  const team = await notion.databases.query({
    database_id: teamsDbId,
    filter: { property: "Name", title: { equals: teamFilter }}
  });
  teamId = team.results[0]?.id;
} else {
  teamId = currentUser.properties["Primary Team"]?.relation?.[0]?.id;
}

const team = await notion.pages.retrieve({ page_id: teamId });
const teamName = team.properties.Name.title[0].plain_text;

const since = new Date();
since.setDate(since.getDate() - period);
const sinceStr = since.toISOString().split('T')[0];

// Get team members
const members = await notion.databases.query({
  database_id: usersDbId,
  filter: { property: "Primary Team", relation: { contains: teamId }}
});

// Get all tasks for period
const allTasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Team", relation: { contains: teamId }},
      { property: "Date", date: { on_or_after: sinceStr }}
    ]
  }
});

// Get completed tasks
const completedTasks = allTasks.results.filter(t =>
  t.properties.Status?.select?.name === "Done"
);

// Get blocked tasks
const blockedTasks = allTasks.results.filter(t =>
  t.properties.Status?.select?.name === "Blocked"
);

// Calculate per-member stats
const memberStats = [];
for (const member of members.results) {
  const memberName = member.properties.Name.title[0].plain_text;
  const memberTasks = allTasks.results.filter(t =>
    t.properties["Assigned To"]?.relation?.[0]?.id === member.id
  );
  const memberCompleted = memberTasks.filter(t =>
    t.properties.Status?.select?.name === "Done"
  );

  memberStats.push({
    name: memberName,
    total: memberTasks.length,
    completed: memberCompleted.length,
    completion: memberTasks.length > 0 ?
      Math.round((memberCompleted.length / memberTasks.length) * 100) : 0
  });
}

// Sort by completion rate
memberStats.sort((a, b) => b.completion - a.completion);

// Team metrics
const teamCompletion = allTasks.results.length > 0 ?
  Math.round((completedTasks.length / allTasks.results.length) * 100) : 0;

// Priority breakdown
const priorityBreakdown = {
  "Critical": allTasks.results.filter(t => t.properties.Priority?.select?.name === "Critical").length,
  "High": allTasks.results.filter(t => t.properties.Priority?.select?.name === "High").length,
  "Medium": allTasks.results.filter(t => t.properties.Priority?.select?.name === "Medium").length,
  "Low": allTasks.results.filter(t => t.properties.Priority?.select?.name === "Low").length
};

// Create report page
const report = await notion.pages.create({
  parent: { page_id: workspaceId },
  properties: {
    title: { title: [{ text: { content: `${teamName} - Performance Report (${sinceStr} to Today)` }}] }
  },
  children: [
    {
      type: "heading_1",
      heading_1: { rich_text: [{ text: { content: `📊 ${teamName} Performance Report` }}] }
    },
    {
      type: "callout",
      callout: {
        rich_text: [{ text: { content: `Period: ${sinceStr} to ${new Date().toISOString().split('T')[0]} (${period} days)` }}],
        icon: { emoji: "📅" }
      }
    },
    {
      type: "heading_2",
      heading_2: { rich_text: [{ text: { content: "📈 Team Metrics" }}] }
    },
    {
      type: "paragraph",
      paragraph: {
        rich_text: [{ text: { content: `
Team Size: ${members.results.length} members
Total Tasks: ${allTasks.results.length}
Completed: ${completedTasks.length} (${teamCompletion}%)
Blocked: ${blockedTasks.length}
` }}]
      }
    },
    {
      type: "heading_2",
      heading_2: { rich_text: [{ text: { content: "👥 Individual Performance" }}] }
    },
    {
      type: "table",
      table: {
        table_width: 4,
        has_column_header: true,
        has_row_header: false,
        children: [
          {
            type: "table_row",
            table_row: {
              cells: [
                [{ type: "text", text: { content: "Member" }}],
                [{ type: "text", text: { content: "Total" }}],
                [{ type: "text", text: { content: "Completed" }}],
                [{ type: "text", text: { content: "Rate" }}]
              ]
            }
          },
          ...memberStats.map(m => ({
            type: "table_row",
            table_row: {
              cells: [
                [{ type: "text", text: { content: m.name }}],
                [{ type: "text", text: { content: String(m.total) }}],
                [{ type: "text", text: { content: String(m.completed) }}],
                [{ type: "text", text: { content: `${m.completion}%` }}]
              ]
            }
          }))
        ]
      }
    }
  ]
});

return `
📊 ${teamName} - Performance Report

📅 Period: Last ${period} days

📈 TEAM METRICS
├── Team Size: ${members.results.length} members
├── Total Tasks: ${allTasks.results.length}
├── Completed: ${completedTasks.length} (${teamCompletion}%)
├── Blocked: ${blockedTasks.length}
└── Completion Rate: ${teamCompletion}%
    ${"▰".repeat(Math.floor(teamCompletion/10))}${"▱".repeat(10-Math.floor(teamCompletion/10))}

📋 PRIORITY BREAKDOWN
├── Critical: ${priorityBreakdown["Critical"]}
├── High: ${priorityBreakdown["High"]}
├── Medium: ${priorityBreakdown["Medium"]}
└── Low: ${priorityBreakdown["Low"]}

👥 TOP PERFORMERS
${memberStats.slice(0, 5).map((m, i) =>
  `${i + 1}. ${m.name}: ${m.completion}% (${m.completed}/${m.total})`
).join('\n')}

Full report generated: View in Notion
`;
```

## Department Report

### Generate Department Report

```
/10x-analytics dept-report --department "Engineering"
```

**Implementation:**

```javascript
const deptName = $ARGUMENTS.match(/--department\s+"([^"]+)"/)?.[1];
const period = parseInt($ARGUMENTS.match(/--period\s+(\d+)/)?.[1]) || 30;

const dept = await notion.databases.query({
  database_id: departmentsDbId,
  filter: { property: "Name", title: { equals: deptName }}
});

if (!dept.results.length) {
  return `Department "${deptName}" not found`;
}

const deptId = dept.results[0].id;
const deptData = dept.results[0];

const since = new Date();
since.setDate(since.getDate() - period);
const sinceStr = since.toISOString().split('T')[0];

// Get teams in department
const teams = await notion.databases.query({
  database_id: teamsDbId,
  filter: { property: "Department", relation: { contains: deptId }}
});

// Get members in department
const members = await notion.databases.query({
  database_id: usersDbId,
  filter: { property: "Department", relation: { contains: deptId }}
});

// Get tasks for department
const tasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Department", relation: { contains: deptId }},
      { property: "Date", date: { on_or_after: sinceStr }}
    ]
  }
});

// Get goals for department
const goals = await notion.databases.query({
  database_id: goalsDbId,
  filter: { property: "Department", relation: { contains: deptId }}
});

const completedTasks = tasks.results.filter(t =>
  t.properties.Status?.select?.name === "Done"
);

const activeGoals = goals.results.filter(g =>
  g.properties.Status?.select?.name === "In Progress"
);

const avgGoalProgress = activeGoals.length > 0 ?
  Math.round(activeGoals.reduce((sum, g) =>
    sum + (g.properties.Progress?.number || 0), 0
  ) / activeGoals.length) : 0;

// Team breakdown
const teamStats = [];
for (const team of teams.results) {
  const teamTasks = tasks.results.filter(t =>
    t.properties.Team?.relation?.[0]?.id === team.id
  );
  const teamCompleted = teamTasks.filter(t =>
    t.properties.Status?.select?.name === "Done"
  );

  teamStats.push({
    name: team.properties.Name.title[0].plain_text,
    tasks: teamTasks.length,
    completed: teamCompleted.length,
    rate: teamTasks.length > 0 ?
      Math.round((teamCompleted.length / teamTasks.length) * 100) : 0
  });
}

const budget = deptData.properties.Budget?.number || 0;

return `
📊 ${deptName} Department Report

📅 Period: Last ${period} days

🏛️ DEPARTMENT OVERVIEW
├── Teams: ${teams.results.length}
├── Members: ${members.results.length}
├── Budget: $${budget.toLocaleString()}
└── Headcount: ${members.results.length}/${deptData.properties["Headcount Limit"]?.number || "∞"}

📈 PERFORMANCE
├── Total Tasks: ${tasks.results.length}
├── Completed: ${completedTasks.length} (${tasks.results.length > 0 ? Math.round((completedTasks.length / tasks.results.length) * 100) : 0}%)
├── Active Goals: ${activeGoals.length}
└── Avg Goal Progress: ${avgGoalProgress}%

👥 TEAM BREAKDOWN
${teamStats.map(t =>
  `├── ${t.name}: ${t.rate}% completion (${t.completed}/${t.tasks})`
).join('\n')}

📋 Generated: ${new Date().toLocaleString()}
`;
```

## Organization Report

### Generate Organization-wide Report

```
/10x-analytics org-report
/10x-analytics org-report --period 90
```

**Implementation:**

```javascript
const period = parseInt($ARGUMENTS.match(/--period\s+(\d+)/)?.[1]) || 30;

const since = new Date();
since.setDate(since.getDate() - period);
const sinceStr = since.toISOString().split('T')[0];

// Get organization
const org = await notion.databases.query({
  database_id: organizationsDbId,
  page_size: 1
});

const orgName = org.results[0].properties.Name.title[0].plain_text;

// Get all departments
const departments = await notion.databases.query({
  database_id: departmentsDbId
});

// Get all teams
const teams = await notion.databases.query({
  database_id: teamsDbId
});

// Get all active users
const users = await notion.databases.query({
  database_id: usersDbId,
  filter: { property: "Status", select: { equals: "Active" }}
});

// Get all tasks
const tasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: { property: "Date", date: { on_or_after: sinceStr }}
});

// Get all goals
const goals = await notion.databases.query({
  database_id: goalsDbId,
  filter: { property: "Status", select: { does_not_equal: "Abandoned" }}
});

// Get RACI assignments
const raciAssignments = await notion.databases.query({
  database_id: raciAssignmentsDbId,
  filter: { property: "Assignment Date", date: { on_or_after: sinceStr }}
});

// Calculate metrics
const completedTasks = tasks.results.filter(t =>
  t.properties.Status?.select?.name === "Done"
);

const completedGoals = goals.results.filter(g =>
  g.properties.Status?.select?.name === "Completed"
);

const activeGoals = goals.results.filter(g =>
  g.properties.Status?.select?.name === "In Progress"
);

const avgGoalProgress = activeGoals.length > 0 ?
  Math.round(activeGoals.reduce((sum, g) =>
    sum + (g.properties.Progress?.number || 0), 0
  ) / activeGoals.length) : 0;

const completedRaci = raciAssignments.results.filter(r =>
  r.properties.Status?.select?.name === "Completed"
);

const taskCompletionRate = tasks.results.length > 0 ?
  Math.round((completedTasks.length / tasks.results.length) * 100) : 0;

const raciComplianceRate = raciAssignments.results.length > 0 ?
  Math.round((completedRaci.length / raciAssignments.results.length) * 100) : 0;

// Department performance
const deptStats = [];
for (const dept of departments.results) {
  const deptTasks = tasks.results.filter(t =>
    t.properties.Department?.relation?.[0]?.id === dept.id
  );
  const deptCompleted = deptTasks.filter(t =>
    t.properties.Status?.select?.name === "Done"
  );

  deptStats.push({
    name: dept.properties.Name.title[0].plain_text,
    rate: deptTasks.length > 0 ?
      Math.round((deptCompleted.length / deptTasks.length) * 100) : 0
  });
}

deptStats.sort((a, b) => b.rate - a.rate);

return `
📊 ${orgName} - Organization Report

📅 Period: Last ${period} days
Generated: ${new Date().toLocaleString()}

🏢 ORGANIZATION OVERVIEW
├── Departments: ${departments.results.length}
├── Teams: ${teams.results.length}
├── Active Users: ${users.results.length}
└── Plan: ${org.results[0].properties.Plan?.select?.name || "N/A"}

📈 KEY METRICS
┌─────────────────────────────────────────┐
│ Task Completion Rate: ${taskCompletionRate}%
│ ${"▰".repeat(Math.floor(taskCompletionRate/10))}${"▱".repeat(10-Math.floor(taskCompletionRate/10))}
│
│ Goal Achievement: ${avgGoalProgress}%
│ ${"▰".repeat(Math.floor(avgGoalProgress/10))}${"▱".repeat(10-Math.floor(avgGoalProgress/10))}
│
│ RACI Compliance: ${raciComplianceRate}%
│ ${"▰".repeat(Math.floor(raciComplianceRate/10))}${"▱".repeat(10-Math.floor(raciComplianceRate/10))}
└─────────────────────────────────────────┘

📋 TASK SUMMARY
├── Total: ${tasks.results.length}
├── Completed: ${completedTasks.length}
├── In Progress: ${tasks.results.filter(t => t.properties.Status?.select?.name === "In Progress").length}
└── Blocked: ${tasks.results.filter(t => t.properties.Status?.select?.name === "Blocked").length}

🎯 GOALS SUMMARY
├── Total: ${goals.results.length}
├── Completed: ${completedGoals.length}
├── In Progress: ${activeGoals.length}
└── Average Progress: ${avgGoalProgress}%

🏆 DEPARTMENT RANKING
${deptStats.map((d, i) =>
  `${i + 1}. ${d.name}: ${d.rate}%`
).join('\n')}

📋 RACI ASSIGNMENTS
├── Total: ${raciAssignments.results.length}
├── Completed: ${completedRaci.length}
└── Compliance Rate: ${raciComplianceRate}%
`;
```

## RACI Compliance Report

### Check RACI Coverage

```
/10x-analytics raci-compliance
```

**Implementation:**

```javascript
// Get all active tasks
const tasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Status", select: { does_not_equal: "Done" }},
      { property: "Status", select: { does_not_equal: "Blocked" }}
    ]
  }
});

// Check RACI coverage for each task
const coverage = {
  full: [],      // Has R, A, C, and I
  partial: [],   // Has some but not all
  none: []       // No RACI assignments
};

for (const task of tasks.results) {
  const taskId = task.id;
  const taskName = task.properties["Task Name"].title[0].plain_text;

  const raciAssignments = await notion.databases.query({
    database_id: raciAssignmentsDbId,
    filter: { property: "Task", relation: { contains: taskId }}
  });

  const types = new Set(raciAssignments.results.map(r =>
    r.properties["RACI Type"].select.name.charAt(0)
  ));

  if (types.size === 0) {
    coverage.none.push(taskName);
  } else if (types.has('R') && types.has('A')) {
    coverage.full.push(taskName);
  } else {
    coverage.partial.push({
      name: taskName,
      has: Array.from(types).join(', '),
      missing: ['R', 'A', 'C', 'I'].filter(t => !types.has(t)).join(', ')
    });
  }
}

// Check for tasks with multiple Accountable
const multipleAccountable = [];
for (const task of tasks.results) {
  const accountable = await notion.databases.query({
    database_id: raciAssignmentsDbId,
    filter: {
      and: [
        { property: "Task", relation: { contains: task.id }},
        { property: "RACI Type", select: { equals: "A - Accountable" }}
      ]
    }
  });

  if (accountable.results.length > 1) {
    multipleAccountable.push(task.properties["Task Name"].title[0].plain_text);
  }
}

const totalTasks = tasks.results.length;
const fullCoverage = coverage.full.length;
const complianceRate = totalTasks > 0 ?
  Math.round((fullCoverage / totalTasks) * 100) : 0;

return `
📋 RACI Compliance Report

📊 COVERAGE SUMMARY
├── Total Active Tasks: ${totalTasks}
├── Full RACI Coverage: ${fullCoverage} (${complianceRate}%)
├── Partial Coverage: ${coverage.partial.length}
└── No Coverage: ${coverage.none.length}

Compliance Rate: ${complianceRate}%
${"▰".repeat(Math.floor(complianceRate/10))}${"▱".repeat(10-Math.floor(complianceRate/10))}

⚠️ ISSUES FOUND

${multipleAccountable.length > 0 ? `
🔴 Multiple Accountable (RACI Violation):
${multipleAccountable.map(t => `  • ${t}`).join('\n')}
` : "✅ No multiple Accountable violations"}

${coverage.none.length > 0 ? `
🟡 Tasks Without RACI:
${coverage.none.slice(0, 10).map(t => `  • ${t}`).join('\n')}
${coverage.none.length > 10 ? `  ... and ${coverage.none.length - 10} more` : ""}
` : "✅ All tasks have RACI assignments"}

${coverage.partial.length > 0 ? `
🟠 Partial RACI Coverage:
${coverage.partial.slice(0, 5).map(t =>
  `  • ${t.name}: Has [${t.has}], Missing [${t.missing}]`
).join('\n')}
${coverage.partial.length > 5 ? `  ... and ${coverage.partial.length - 5} more` : ""}
` : ""}

📝 RECOMMENDATIONS
${complianceRate < 50 ? "• Critical: Most tasks lack proper RACI assignments" : ""}
${multipleAccountable.length > 0 ? "• Fix tasks with multiple Accountable persons" : ""}
${coverage.none.length > 5 ? "• Assign RACI roles to tasks without coverage" : ""}
${complianceRate >= 80 ? "• Great job! RACI compliance is healthy" : ""}
`;
```

## KPI Dashboard

### View Key Performance Indicators

```
/10x-analytics kpi
/10x-analytics kpi --period 30
```

**Implementation:**

```javascript
const period = parseInt($ARGUMENTS.match(/--period\s+(\d+)/)?.[1]) || 30;

const since = new Date();
since.setDate(since.getDate() - period);
const sinceStr = since.toISOString().split('T')[0];

// Calculate all KPIs
const tasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: { property: "Date", date: { on_or_after: sinceStr }}
});

const completedTasks = tasks.results.filter(t =>
  t.properties.Status?.select?.name === "Done"
);

const goals = await notion.databases.query({
  database_id: goalsDbId,
  filter: {
    and: [
      { property: "Status", select: { equals: "In Progress" }},
      { property: "Target Date", date: { on_or_after: sinceStr }}
    ]
  }
});

const users = await notion.databases.query({
  database_id: usersDbId,
  filter: { property: "Status", select: { equals: "Active" }}
});

// KPI Calculations
const kpis = {
  taskCompletionRate: tasks.results.length > 0 ?
    Math.round((completedTasks.length / tasks.results.length) * 100) : 0,

  avgTasksPerUser: users.results.length > 0 ?
    Math.round(tasks.results.length / users.results.length) : 0,

  goalsOnTrack: goals.results.filter(g => {
    const progress = g.properties.Progress?.number || 0;
    const start = new Date(g.properties["Start Date"]?.date?.start);
    const target = new Date(g.properties["Target Date"]?.date?.start);
    const expected = ((new Date() - start) / (target - start)) * 100;
    return progress >= expected - 10;
  }).length,

  avgGoalProgress: goals.results.length > 0 ?
    Math.round(goals.results.reduce((sum, g) =>
      sum + (g.properties.Progress?.number || 0), 0
    ) / goals.results.length) : 0,

  blockedTaskRate: tasks.results.length > 0 ?
    Math.round((tasks.results.filter(t =>
      t.properties.Status?.select?.name === "Blocked"
    ).length / tasks.results.length) * 100) : 0
};

// Trend indicators (compare to previous period)
const prevSince = new Date(since);
prevSince.setDate(prevSince.getDate() - period);
const prevSinceStr = prevSince.toISOString().split('T')[0];

const prevTasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Date", date: { on_or_after: prevSinceStr }},
      { property: "Date", date: { before: sinceStr }}
    ]
  }
});

const prevCompleted = prevTasks.results.filter(t =>
  t.properties.Status?.select?.name === "Done"
);

const prevCompletionRate = prevTasks.results.length > 0 ?
  Math.round((prevCompleted.length / prevTasks.results.length) * 100) : 0;

const trend = kpis.taskCompletionRate > prevCompletionRate ? "📈" :
  kpis.taskCompletionRate < prevCompletionRate ? "📉" : "➡️";

return `
📊 KPI Dashboard

📅 Period: Last ${period} days
Generated: ${new Date().toLocaleString()}

┌─────────────────────────────────────────────────────────┐
│                    KEY PERFORMANCE INDICATORS            │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  📈 Task Completion Rate: ${kpis.taskCompletionRate}% ${trend}
│     ${"▰".repeat(Math.floor(kpis.taskCompletionRate/10))}${"▱".repeat(10-Math.floor(kpis.taskCompletionRate/10))}
│     Previous period: ${prevCompletionRate}%
│                                                          │
│  🎯 Goals On Track: ${kpis.goalsOnTrack}/${goals.results.length}
│     Average Progress: ${kpis.avgGoalProgress}%
│     ${"▰".repeat(Math.floor(kpis.avgGoalProgress/10))}${"▱".repeat(10-Math.floor(kpis.avgGoalProgress/10))}
│                                                          │
│  👥 Tasks per User: ${kpis.avgTasksPerUser}
│     Active Users: ${users.results.length}
│                                                          │
│  🚫 Blocked Task Rate: ${kpis.blockedTaskRate}%
│     ${kpis.blockedTaskRate > 10 ? "⚠️ High - needs attention" : "✅ Healthy"}
│                                                          │
└─────────────────────────────────────────────────────────┘

📋 SUMMARY
${kpis.taskCompletionRate >= 80 ? "✅ Excellent task completion rate" : kpis.taskCompletionRate >= 60 ? "🟡 Good task completion, room for improvement" : "🔴 Task completion needs attention"}
${kpis.goalsOnTrack >= goals.results.length * 0.7 ? "✅ Most goals on track" : "⚠️ Several goals at risk"}
${kpis.blockedTaskRate <= 5 ? "✅ Low blocked task rate" : "⚠️ High number of blocked tasks"}
`;
```

## Export Reports

### Export to Various Formats

```
/10x-analytics export --format json
/10x-analytics export --format csv --type tasks
```

## Troubleshooting

**"Permission denied for org report"**
→ Organization reports require Admin access
→ Contact an Admin for access

**"No data found for period"**
→ Try extending the period with --period flag
→ Verify tasks/goals have dates set

**"Report generation failed"**
→ Check Notion MCP server connection
→ Verify database access permissions

## Version

Skill Version: 2.0.0
Required Role: Varies by report type
Compatible with: Notion MCP Server 1.0+
