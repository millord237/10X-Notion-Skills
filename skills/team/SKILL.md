---
name: 10x-team
description: Team operations including standups, sprint planning, team boards, and capacity planning. Use for running daily standups, planning sprints, viewing team workload, and managing team activities. Requires Team Lead or higher access.
allowed-tools: Read, Write, Bash
user-invocable: true
argument-hint: "[command] [args]"
---

# 10X Notion Team Skills - Team Operations

Manage team activities, run standups, plan sprints, and track team performance.

## Quick Commands

| Command | Description | Role Required |
|---------|-------------|---------------|
| `board` | View team kanban board | Member |
| `standup` | Run daily standup | Team Lead |
| `sprint` | Sprint planning/management | Team Lead |
| `capacity` | View team capacity | Team Lead |
| `workload` | View member workload | Team Lead |
| `velocity` | View team velocity | Team Lead |

## Team Board

### View Team Kanban Board

```
/10x-team board
/10x-team board --team "Backend Team"
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();
const teamFilter = $ARGUMENTS.match(/--team\s+"([^"]+)"/)?.[1];

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

if (!teamId) {
  return "No team specified or you're not assigned to a team";
}

// Get team details
const team = await notion.pages.retrieve({ page_id: teamId });
const teamName = team.properties.Name.title[0].plain_text;

// Get all team tasks
const tasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: { property: "Team", relation: { contains: teamId }},
  sorts: [
    { property: "Priority", direction: "ascending" },
    { property: "Date", direction: "ascending" }
  ]
});

// Group by status
const board = {
  "To Do": [],
  "In Progress": [],
  "Blocked": [],
  "Done": []
};

for (const task of tasks.results) {
  const status = task.properties.Status.select?.name || "To Do";
  const assignee = task.properties["Assigned To"]?.relation?.[0];
  let assigneeName = "Unassigned";

  if (assignee) {
    const user = await notion.pages.retrieve({ page_id: assignee.id });
    assigneeName = user.properties.Name.title[0].plain_text;
  }

  if (board[status]) {
    board[status].push({
      name: task.properties["Task Name"].title[0].plain_text,
      assignee: assigneeName,
      priority: task.properties.Priority?.select?.name || "Medium",
      dueDate: task.properties.Date?.date?.start
    });
  }
}

return `
📋 ${teamName} - Team Board

📥 TO DO (${board["To Do"].length})
${board["To Do"].map(t => `  • [${t.priority}] ${t.name} → ${t.assignee}`).join('\n') || "  Empty"}

🔄 IN PROGRESS (${board["In Progress"].length})
${board["In Progress"].map(t => `  • [${t.priority}] ${t.name} → ${t.assignee}`).join('\n') || "  Empty"}

🚫 BLOCKED (${board["Blocked"].length})
${board["Blocked"].map(t => `  • [${t.priority}] ${t.name} → ${t.assignee}`).join('\n') || "  None"}

✅ DONE (${board["Done"].length})
${board["Done"].slice(0, 5).map(t => `  • ${t.name} → ${t.assignee}`).join('\n') || "  Empty"}
${board["Done"].length > 5 ? `  ... and ${board["Done"].length - 5} more` : ""}

Total: ${tasks.results.length} tasks
`;
```

## Daily Standup

### Run Standup

```
/10x-team standup
/10x-team standup --team "Backend Team"
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();
const teamId = currentUser.properties["Primary Team"]?.relation?.[0]?.id;

if (!teamId) {
  return "You must be assigned to a team to run standup";
}

// Check if user is team lead
const team = await notion.pages.retrieve({ page_id: teamId });
const teamLeadId = team.properties["Team Lead"]?.relation?.[0]?.id;

if (teamLeadId !== currentUser.id) {
  // Check if user has admin role
  const hasPermission = await checkPermission("manage_teams", currentUser.id);
  if (!hasPermission) {
    return "Only the Team Lead can run standups";
  }
}

const teamName = team.properties.Name.title[0].plain_text;

// Get team members
const members = await notion.databases.query({
  database_id: usersDbId,
  filter: {
    and: [
      { property: "Primary Team", relation: { contains: teamId }},
      { property: "Status", select: { equals: "Active" }}
    ]
  }
});

// Get yesterday's date
const yesterday = new Date();
yesterday.setDate(yesterday.getDate() - 1);
const yesterdayStr = yesterday.toISOString().split('T')[0];

// Get today's date
const today = new Date().toISOString().split('T')[0];

// Compile standup data for each member
const standupData = [];

for (const member of members.results) {
  const memberName = member.properties.Name.title[0].plain_text;

  // Tasks completed yesterday
  const completedYesterday = await notion.databases.query({
    database_id: tasksDbId,
    filter: {
      and: [
        { property: "Assigned To", relation: { contains: member.id }},
        { property: "Status", select: { equals: "Done" }},
        { property: "Date", date: { equals: yesterdayStr }}
      ]
    }
  });

  // Tasks in progress today
  const inProgress = await notion.databases.query({
    database_id: tasksDbId,
    filter: {
      and: [
        { property: "Assigned To", relation: { contains: member.id }},
        { property: "Status", select: { equals: "In Progress" }}
      ]
    }
  });

  // Blocked tasks
  const blocked = await notion.databases.query({
    database_id: tasksDbId,
    filter: {
      and: [
        { property: "Assigned To", relation: { contains: member.id }},
        { property: "Status", select: { equals: "Blocked" }}
      ]
    }
  });

  standupData.push({
    name: memberName,
    completedYesterday: completedYesterday.results.map(t =>
      t.properties["Task Name"].title[0].plain_text
    ),
    workingOn: inProgress.results.map(t =>
      t.properties["Task Name"].title[0].plain_text
    ),
    blockers: blocked.results.map(t =>
      t.properties["Task Name"].title[0].plain_text
    )
  });
}

// Create standup record page
const standupPage = await notion.pages.create({
  parent: { database_id: dailyCheckInsDbId },
  properties: {
    "Date": { date: { start: today }}
  },
  children: [
    {
      type: "heading_1",
      heading_1: {
        rich_text: [{ text: { content: `📢 ${teamName} Daily Standup` }}]
      }
    },
    {
      type: "callout",
      callout: {
        rich_text: [{ text: { content: `Date: ${new Date().toLocaleDateString('en-US', { weekday: 'long', month: 'long', day: 'numeric', year: 'numeric' })}` }}],
        icon: { emoji: "📅" }
      }
    },
    ...standupData.flatMap(member => [
      {
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: `👤 ${member.name}` }}]
        }
      },
      {
        type: "heading_3",
        heading_3: {
          rich_text: [{ text: { content: "✅ Completed Yesterday" }}]
        }
      },
      ...member.completedYesterday.map(task => ({
        type: "bulleted_list_item",
        bulleted_list_item: {
          rich_text: [{ text: { content: task }}]
        }
      })),
      {
        type: "heading_3",
        heading_3: {
          rich_text: [{ text: { content: "🔄 Working On Today" }}]
        }
      },
      ...member.workingOn.map(task => ({
        type: "bulleted_list_item",
        bulleted_list_item: {
          rich_text: [{ text: { content: task }}]
        }
      })),
      ...(member.blockers.length > 0 ? [
        {
          type: "heading_3",
          heading_3: {
            rich_text: [{ text: { content: "🚫 Blockers" }}],
            color: "red_background"
          }
        },
        ...member.blockers.map(task => ({
          type: "bulleted_list_item",
          bulleted_list_item: {
            rich_text: [{ text: { content: task }}]
          }
        }))
      ] : []),
      { type: "divider", divider: {} }
    ])
  ]
});

// Summary
const totalCompleted = standupData.reduce((sum, m) => sum + m.completedYesterday.length, 0);
const totalInProgress = standupData.reduce((sum, m) => sum + m.workingOn.length, 0);
const totalBlockers = standupData.reduce((sum, m) => sum + m.blockers.length, 0);

return `
📢 ${teamName} Daily Standup - ${today}

Team Members: ${members.results.length}
Completed Yesterday: ${totalCompleted} tasks
Working On Today: ${totalInProgress} tasks
${totalBlockers > 0 ? `⚠️ Blockers: ${totalBlockers} tasks need attention` : "✅ No blockers!"}

Standup recorded: View at Notion
`;
```

## Sprint Planning

### Create New Sprint

```
/10x-team sprint new --name "Sprint 23" --start "2025-02-01" --end "2025-02-14"
```

**Implementation:**

```javascript
const sprintName = $ARGUMENTS.match(/--name\s+"([^"]+)"/)?.[1];
const startDate = $ARGUMENTS.match(/--start\s+"([^"]+)"/)?.[1];
const endDate = $ARGUMENTS.match(/--end\s+"([^"]+)"/)?.[1];

const currentUser = await getCurrentUser();
const teamId = currentUser.properties["Primary Team"]?.relation?.[0]?.id;
const team = await notion.pages.retrieve({ page_id: teamId });

// Get team capacity
const members = await notion.databases.query({
  database_id: usersDbId,
  filter: { property: "Primary Team", relation: { contains: teamId }}
});

const capacity = team.properties["Capacity Points"]?.number || (members.results.length * 10);

// Create sprint as a project
const sprint = await notion.pages.create({
  parent: { database_id: projectsDbId },
  properties: {
    "Name": { title: [{ text: { content: sprintName }}] },
    "Status": { select: { name: "Planning" }},
    "Start Date": { date: { start: startDate }},
    "End Date": { date: { start: endDate }},
    "Team": { relation: [{ id: teamId }] },
    "Project Manager": { relation: [{ id: currentUser.id }] }
  },
  children: [
    {
      type: "heading_1",
      heading_1: {
        rich_text: [{ text: { content: `🏃 ${sprintName}` }}]
      }
    },
    {
      type: "callout",
      callout: {
        rich_text: [{ text: { content: `${startDate} → ${endDate} | Capacity: ${capacity} points` }}],
        icon: { emoji: "📊" }
      }
    },
    {
      type: "heading_2",
      heading_2: {
        rich_text: [{ text: { content: "🎯 Sprint Goals" }}]
      }
    },
    {
      type: "numbered_list_item",
      numbered_list_item: { rich_text: [] }
    },
    {
      type: "heading_2",
      heading_2: {
        rich_text: [{ text: { content: "📋 Sprint Backlog" }}]
      }
    },
    {
      type: "paragraph",
      paragraph: {
        rich_text: [{ text: { content: "Add tasks to this sprint using /10x-team sprint add-task" }}]
      }
    }
  ]
});

return `
🏃 Sprint Created!

Name: ${sprintName}
Duration: ${startDate} → ${endDate}
Team: ${team.properties.Name.title[0].plain_text}
Capacity: ${capacity} points

Next steps:
1. Add tasks: /10x-team sprint add-task "Task name" --points 3
2. Assign tasks: /10x-delegate "Task name" --to "User"
3. Start sprint: /10x-team sprint start
`;
```

### Add Task to Sprint

```
/10x-team sprint add-task "Implement login" --points 5 --assignee "John"
```

### View Sprint Progress

```
/10x-team sprint status
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();
const teamId = currentUser.properties["Primary Team"]?.relation?.[0]?.id;

// Get current sprint (most recent active project for team)
const sprint = await notion.databases.query({
  database_id: projectsDbId,
  filter: {
    and: [
      { property: "Team", relation: { contains: teamId }},
      { property: "Status", select: { equals: "In Progress" }}
    ]
  },
  sorts: [{ property: "Start Date", direction: "descending" }],
  page_size: 1
});

if (!sprint.results.length) {
  return "No active sprint found. Create one with /10x-team sprint new";
}

const sprintData = sprint.results[0];
const sprintName = sprintData.properties.Name.title[0].plain_text;
const startDate = sprintData.properties["Start Date"]?.date?.start;
const endDate = sprintData.properties["End Date"]?.date?.start;

// Get sprint tasks
const tasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: { property: "Related Project", relation: { contains: sprintData.id }}
});

// Calculate metrics
const statusCounts = { "To Do": 0, "In Progress": 0, "Done": 0, "Blocked": 0 };
let totalPoints = 0;
let completedPoints = 0;

for (const task of tasks.results) {
  const status = task.properties.Status.select?.name || "To Do";
  statusCounts[status] = (statusCounts[status] || 0) + 1;

  const points = task.properties["Story Points"]?.number || 1;
  totalPoints += points;
  if (status === "Done") {
    completedPoints += points;
  }
}

// Calculate sprint progress
const today = new Date();
const start = new Date(startDate);
const end = new Date(endDate);
const totalDays = (end - start) / (1000 * 60 * 60 * 24);
const elapsedDays = Math.max(0, (today - start) / (1000 * 60 * 60 * 24));
const timeProgress = Math.min(100, Math.round((elapsedDays / totalDays) * 100));
const workProgress = totalPoints > 0 ? Math.round((completedPoints / totalPoints) * 100) : 0;

// Burndown status
let burndownStatus;
if (workProgress >= timeProgress) {
  burndownStatus = "🟢 On Track";
} else if (workProgress >= timeProgress - 10) {
  burndownStatus = "🟡 Slightly Behind";
} else {
  burndownStatus = "🔴 At Risk";
}

return `
🏃 ${sprintName} - Sprint Status

📅 Timeline: ${startDate} → ${endDate}
   Time Progress: ${timeProgress}% (${"▰".repeat(Math.floor(timeProgress/10))}${"▱".repeat(10-Math.floor(timeProgress/10))})

📊 Work Progress: ${workProgress}% (${completedPoints}/${totalPoints} points)
   ${"▰".repeat(Math.floor(workProgress/10))}${"▱".repeat(10-Math.floor(workProgress/10))}

📋 Task Breakdown:
   • To Do: ${statusCounts["To Do"]}
   • In Progress: ${statusCounts["In Progress"]}
   • Done: ${statusCounts["Done"]}
   • Blocked: ${statusCounts["Blocked"]}

${burndownStatus} - Sprint ${workProgress >= timeProgress ? "ahead of" : "behind"} schedule
`;
```

## Capacity Planning

### View Team Capacity

```
/10x-team capacity
```

**Implementation:**

```javascript
const currentUser = await getCurrentUser();
const teamId = currentUser.properties["Primary Team"]?.relation?.[0]?.id;
const team = await notion.pages.retrieve({ page_id: teamId });
const teamName = team.properties.Name.title[0].plain_text;

// Get team members
const members = await notion.databases.query({
  database_id: usersDbId,
  filter: {
    and: [
      { property: "Primary Team", relation: { contains: teamId }},
      { property: "Status", select: { equals: "Active" }}
    ]
  }
});

const capacityData = [];
let totalCapacity = 0;
let totalAssigned = 0;

for (const member of members.results) {
  const memberName = member.properties.Name.title[0].plain_text;

  // Get assigned tasks (not done)
  const tasks = await notion.databases.query({
    database_id: tasksDbId,
    filter: {
      and: [
        { property: "Assigned To", relation: { contains: member.id }},
        { property: "Status", select: { does_not_equal: "Done" }}
      ]
    }
  });

  // Calculate points assigned
  let pointsAssigned = 0;
  for (const task of tasks.results) {
    pointsAssigned += task.properties["Story Points"]?.number || 1;
  }

  // Assume 10 points capacity per person per sprint
  const capacity = 10;
  const utilization = Math.round((pointsAssigned / capacity) * 100);

  capacityData.push({
    name: memberName,
    capacity,
    assigned: pointsAssigned,
    available: Math.max(0, capacity - pointsAssigned),
    utilization,
    taskCount: tasks.results.length
  });

  totalCapacity += capacity;
  totalAssigned += pointsAssigned;
}

// Sort by available capacity
capacityData.sort((a, b) => b.available - a.available);

const teamUtilization = Math.round((totalAssigned / totalCapacity) * 100);

return `
📊 ${teamName} - Capacity Planning

Team Utilization: ${teamUtilization}%
${"▰".repeat(Math.floor(teamUtilization/10))}${"▱".repeat(10-Math.floor(teamUtilization/10))}

┌─────────────────┬──────────┬──────────┬───────────┬─────────────┐
│ Member          │ Capacity │ Assigned │ Available │ Utilization │
├─────────────────┼──────────┼──────────┼───────────┼─────────────┤
${capacityData.map(m =>
  `│ ${m.name.padEnd(15)} │ ${String(m.capacity).padStart(8)} │ ${String(m.assigned).padStart(8)} │ ${String(m.available).padStart(9)} │ ${(m.utilization + "%").padStart(11)} │`
).join('\n')}
└─────────────────┴──────────┴──────────┴───────────┴─────────────┘

Total: ${totalAssigned}/${totalCapacity} points assigned
Available capacity: ${totalCapacity - totalAssigned} points

${teamUtilization > 100 ? "⚠️ Team is overloaded!" : teamUtilization > 80 ? "🟡 Team is near capacity" : "🟢 Capacity available"}
`;
```

### View Member Workload

```
/10x-team workload "John Doe"
/10x-team workload --all
```

## Team Velocity

### View Velocity Trend

```
/10x-team velocity
/10x-team velocity --sprints 5
```

**Implementation:**

```javascript
const sprintCount = parseInt($ARGUMENTS.match(/--sprints\s+(\d+)/)?.[1]) || 5;
const currentUser = await getCurrentUser();
const teamId = currentUser.properties["Primary Team"]?.relation?.[0]?.id;
const team = await notion.pages.retrieve({ page_id: teamId });
const teamName = team.properties.Name.title[0].plain_text;

// Get completed sprints
const sprints = await notion.databases.query({
  database_id: projectsDbId,
  filter: {
    and: [
      { property: "Team", relation: { contains: teamId }},
      { property: "Status", select: { equals: "Completed" }}
    ]
  },
  sorts: [{ property: "End Date", direction: "descending" }],
  page_size: sprintCount
});

const velocityData = [];

for (const sprint of sprints.results) {
  const sprintName = sprint.properties.Name.title[0].plain_text;

  // Get completed tasks for this sprint
  const tasks = await notion.databases.query({
    database_id: tasksDbId,
    filter: {
      and: [
        { property: "Related Project", relation: { contains: sprint.id }},
        { property: "Status", select: { equals: "Done" }}
      ]
    }
  });

  let points = 0;
  for (const task of tasks.results) {
    points += task.properties["Story Points"]?.number || 1;
  }

  velocityData.push({
    sprint: sprintName,
    points,
    tasks: tasks.results.length
  });
}

// Calculate average velocity
const avgVelocity = velocityData.length > 0 ?
  Math.round(velocityData.reduce((sum, v) => sum + v.points, 0) / velocityData.length) : 0;

// Velocity trend
const recentAvg = velocityData.slice(0, 3).reduce((sum, v) => sum + v.points, 0) / Math.min(3, velocityData.length);
const olderAvg = velocityData.slice(3).reduce((sum, v) => sum + v.points, 0) / Math.max(1, velocityData.length - 3);
const trend = recentAvg > olderAvg ? "📈 Improving" : recentAvg < olderAvg ? "📉 Declining" : "➡️ Stable";

return `
📈 ${teamName} - Velocity Trend

Average Velocity: ${avgVelocity} points/sprint
Trend: ${trend}

Sprint History:
${velocityData.map(v =>
  `  ${v.sprint}: ${v.points} pts (${"■".repeat(Math.min(20, Math.floor(v.points/2)))})`
).join('\n')}

Recommended Sprint Capacity: ${Math.round(avgVelocity * 0.9)} - ${Math.round(avgVelocity * 1.1)} points
`;
```

## Team Settings

### Update Team Settings

```
/10x-team settings --sprint-length "2 Weeks" --capacity 80
```

## Troubleshooting

**"You must be assigned to a team"**
→ Contact Admin to assign you to a team
→ Use `/10x-admin list-teams` to see available teams

**"Only Team Lead can run standups"**
→ Standups require Team Lead role
→ Ask your Team Lead to run the standup

**"No active sprint found"**
→ Create a sprint with `/10x-team sprint new`

## Version

Skill Version: 2.0.0
Required Role: Team Lead (most commands), Member (board view)
Compatible with: Notion MCP Server 1.0+
