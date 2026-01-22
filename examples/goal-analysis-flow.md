# Goal Analysis Flow

Complete implementation for analyzing goal progress and generating insights.

## Overview

This flow:
- Calculates time-based progress metrics
- Compares actual vs. expected progress
- Determines goal health status
- Provides actionable recommendations
- Queries related action items
- Generates visual progress reports

## Implementation

```javascript
async function analyzeGoalProgress(goalId) {
  // Step 1: Fetch goal details
  const goal = await notion.pages.retrieve({ page_id: goalId });
  const props = goal.properties;
  
  // Step 2: Extract timeline data
  const startDate = new Date(props["Start Date"].date.start);
  const targetDate = new Date(props["Target Date"].date.start);
  const today = new Date();
  
  // Step 3: Calculate time metrics
  const totalDays = Math.floor((targetDate - startDate) / (1000 * 60 * 60 * 24));
  const elapsedDays = Math.floor((today - startDate) / (1000 * 60 * 60 * 24));
  const remainingDays = Math.floor((targetDate - today) / (1000 * 60 * 60 * 24));
  
  const percentTimeElapsed = (elapsedDays / totalDays) * 100;
  const currentProgress = props.Progress.number || 0;
  const expectedProgress = percentTimeElapsed;
  const progressVariance = currentProgress - expectedProgress;
  
  // Step 4: Determine status and recommendations
  let status = {
    onTrack: progressVariance >= -5,
    severity: "normal",
    emoji: "✅",
    recommendation: ""
  };
  
  if (progressVariance >= 10) {
    status.severity = "ahead";
    status.emoji = "🚀";
    status.recommendation = "You're ahead of schedule! Consider:\n" +
      "• Increasing the goal target\n" +
      "• Adding related sub-goals\n" +
      "• Sharing your success with others";
  } else if (progressVariance >= -5 && progressVariance < 10) {
    status.severity = "on-track";
    status.emoji = "✅";
    status.recommendation = "You're on track! Keep up the current pace:\n" +
      "• Maintain consistent effort\n" +
      "• Review weekly progress\n" +
      "• Celebrate small wins";
  } else if (progressVariance >= -15 && progressVariance < -5) {
    status.severity = "at-risk";
    status.emoji = "⚠️";
    status.recommendation = "You're falling behind. Consider:\n" +
      "• Breaking the goal into smaller tasks\n" +
      "• Scheduling dedicated time blocks\n" +
      "• Identifying and removing blockers\n" +
      "• Adjusting timeline if needed";
  } else {
    status.severity = "critical";
    status.emoji = "🔴";
    status.recommendation = "This goal needs immediate attention:\n" +
      "• Review if timeline is realistic\n" +
      "• Identify major obstacles\n" +
      "• Consider breaking into smaller goals\n" +
      "• Reassess priority level\n" +
      "• Seek help or resources";
  }
  
  // Step 5: Calculate required daily progress
  const requiredDailyProgress = remainingDays > 0 ? 
    (100 - currentProgress) / remainingDays : 0;
  
  // Step 6: Fetch related action items
  const actionItems = await notion.databases.query({
    database_id: tasksDbId,
    filter: {
      property: "Related Goal",
      relation: { contains: goalId }
    }
  });
  
  const completedActions = actionItems.results.filter(
    task => task.properties.Status.select?.name === "Done"
  ).length;
  
  const totalActions = actionItems.results.length;
  const actionCompletionRate = totalActions > 0 ? 
    (completedActions / totalActions) * 100 : 0;
  
  const pendingActions = actionItems.results
    .filter(t => t.properties.Status.select?.name !== "Done")
    .map(t => ({
      name: t.properties["Task Name"].title[0].plain_text,
      priority: t.properties.Priority?.select?.name || "Medium",
      dueDate: t.properties.Date?.date?.start
    }))
    .sort((a, b) => {
      const priorityOrder = { "Critical": 0, "High": 1, "Medium": 2, "Low": 3 };
      return priorityOrder[a.priority] - priorityOrder[b.priority];
    });
  
  // Step 7: Query sub-goals if any
  const subGoals = await notion.databases.query({
    database_id: goalsDbId,
    filter: {
      property: "Parent Goal",
      relation: { contains: goalId }
    }
  });
  
  const subGoalProgress = subGoals.results.map(sg => ({
    name: sg.properties.Name.title[0].plain_text,
    progress: sg.properties.Progress.number || 0,
    status: sg.properties.Status.select?.name
  }));
  
  // Step 8: Generate visual progress bar
  const progressBar = generateProgressBar(currentProgress);
  const expectedBar = generateProgressBar(expectedProgress);
  
  // Step 9: Generate report
  const report = {
    goalName: props.Name.title[0].plain_text,
    category: props.Category?.multi_select?.map(c => c.name).join(", ") || "Uncategorized",
    priority: props.Priority?.select?.name || "Medium",
    
    timeline: {
      startDate: startDate.toLocaleDateString(),
      targetDate: targetDate.toLocaleDateString(),
      daysElapsed: elapsedDays,
      daysRemaining: remainingDays,
      totalDays: totalDays,
      percentTimeElapsed: Math.round(percentTimeElapsed)
    },
    
    progress: {
      current: Math.round(currentProgress),
      expected: Math.round(expectedProgress),
      variance: Math.round(progressVariance),
      requiredDailyRate: Math.round(requiredDailyProgress * 10) / 10,
      visualCurrent: progressBar,
      visualExpected: expectedBar
    },
    
    actionItems: {
      completed: completedActions,
      total: totalActions,
      completionRate: Math.round(actionCompletionRate),
      pending: pendingActions.slice(0, 5) // Top 5 pending
    },
    
    subGoals: subGoalProgress,
    
    status: status,
    
    insights: generateInsights(currentProgress, expectedProgress, actionCompletionRate, remainingDays)
  };
  
  return report;
}

function generateProgressBar(progress) {
  const filled = Math.floor(progress / 10);
  const empty = 10 - filled;
  return `[${('█'.repeat(filled))}${'░'.repeat(empty)}] ${Math.round(progress)}%`;
}

function generateInsights(current, expected, actionRate, daysLeft) {
  const insights = [];
  
  // Progress vs. expectations
  if (current >= expected + 10) {
    insights.push("💪 Excellent momentum! You're outpacing the timeline.");
  } else if (current >= expected) {
    insights.push("✅ Solid progress. You're meeting expectations.");
  } else if (current >= expected - 10) {
    insights.push("⚠️ Slightly behind. Small adjustments needed.");
  } else {
    insights.push("🔴 Significant gap. Requires immediate action.");
  }
  
  // Action completion
  if (actionRate >= 80) {
    insights.push("🎯 Strong task completion rate!");
  } else if (actionRate >= 50) {
    insights.push("📋 Moderate task progress. Focus on clearing backlog.");
  } else {
    insights.push("⚡ Low task completion. Break down into smaller actions.");
  }
  
  // Time pressure
  if (daysLeft < 7) {
    insights.push("⏰ Less than a week remaining. Final sprint needed!");
  } else if (daysLeft < 30) {
    insights.push("📅 Less than a month left. Increase intensity.");
  }
  
  // Recommendations based on combination
  if (current < expected && actionRate < 50) {
    insights.push("💡 Tip: Schedule daily time blocks for this goal.");
  }
  
  return insights;
}
```

## Creating Progress Report Page

```javascript
async function createProgressReportPage(goalId, analysis) {
  const reportPage = await notion.pages.create({
    parent: { page_id: workspaceId },
    properties: {
      title: {
        title: [{
          text: {
            content: `Progress Report: ${analysis.goalName}`
          }
        }]
      }
    },
    children: [
      {
        object: "block",
        type: "heading_1",
        heading_1: {
          rich_text: [{
            text: { content: `${analysis.status.emoji} ${analysis.goalName}` }
          }],
          color: getStatusColor(analysis.status.severity)
        }
      },
      
      {
        object: "block",
        type: "callout",
        callout: {
          rich_text: [{
            text: { content: `Status: ${analysis.status.severity.toUpperCase()}` }
          }],
          icon: { emoji: analysis.status.emoji },
          color: getStatusColor(analysis.status.severity)
        }
      },
      
      {
        object: "block",
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: "📊 Progress Overview" }}]
        }
      },
      
      {
        object: "block",
        type: "code",
        code: {
          rich_text: [{
            text: {
              content: `Current:  ${analysis.progress.visualCurrent}\nExpected: ${analysis.progress.visualExpected}\n\nVariance: ${analysis.progress.variance > 0 ? '+' : ''}${analysis.progress.variance}%\nRequired daily rate: ${analysis.progress.requiredDailyRate}%/day`
            }
          }],
          language: "plain text"
        }
      },
      
      {
        object: "block",
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: "📅 Timeline" }}]
        }
      },
      
      {
        object: "block",
        type: "table",
        table: {
          table_width: 2,
          has_column_header: true,
          has_row_header: false,
          children: [
            {
              object: "block",
              type: "table_row",
              table_row: {
                cells: [
                  [{ text: { content: "Metric" }}],
                  [{ text: { content: "Value" }}]
                ]
              }
            },
            {
              object: "block",
              type: "table_row",
              table_row: {
                cells: [
                  [{ text: { content: "Start Date" }}],
                  [{ text: { content: analysis.timeline.startDate }}]
                ]
              }
            },
            {
              object: "block",
              type: "table_row",
              table_row: {
                cells: [
                  [{ text: { content: "Target Date" }}],
                  [{ text: { content: analysis.timeline.targetDate }}]
                ]
              }
            },
            {
              object: "block",
              type: "table_row",
              table_row: {
                cells: [
                  [{ text: { content: "Days Elapsed" }}],
                  [{ text: { content: `${analysis.timeline.daysElapsed} / ${analysis.timeline.totalDays}` }}]
                ]
              }
            },
            {
              object: "block",
              type: "table_row",
              table_row: {
                cells: [
                  [{ text: { content: "Days Remaining" }}],
                  [{ text: { content: analysis.timeline.daysRemaining.toString() }}]
                ]
              }
            }
          ]
        }
      },
      
      {
        object: "block",
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: "✅ Action Items" }}]
        }
      },
      
      {
        object: "block",
        type: "paragraph",
        paragraph: {
          rich_text: [{
            text: {
              content: `Completed: ${analysis.actionItems.completed} / ${analysis.actionItems.total} (${analysis.actionItems.completionRate}%)`
            }
          }]
        }
      },
      
      {
        object: "block",
        type: "heading_3",
        heading_3: {
          rich_text: [{ text: { content: "Top Priority Actions" }}]
        }
      },
      
      ...analysis.actionItems.pending.map(action => ({
        object: "block",
        type: "to_do",
        to_do: {
          rich_text: [{
            text: {
              content: `[${action.priority}] ${action.name}${action.dueDate ? ` - Due: ${action.dueDate}` : ''}`
            }
          }],
          checked: false,
          color: getPriorityColor(action.priority)
        }
      })),
      
      {
        object: "block",
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: "💡 Insights & Recommendations" }}]
        }
      },
      
      ...analysis.insights.map(insight => ({
        object: "block",
        type: "bulleted_list_item",
        bulleted_list_item: {
          rich_text: [{ text: { content: insight }}]
        }
      })),
      
      {
        object: "block",
        type: "divider",
        divider: {}
      },
      
      {
        object: "block",
        type: "callout",
        callout: {
          rich_text: [{
            text: { content: analysis.status.recommendation }
          }],
          icon: { emoji: "🎯" },
          color: "blue_background"
        }
      }
    ]
  });
  
  return reportPage;
}

function getStatusColor(severity) {
  const colors = {
    "ahead": "green_background",
    "on-track": "blue_background",
    "at-risk": "yellow_background",
    "critical": "red_background"
  };
  return colors[severity] || "default";
}

function getPriorityColor(priority) {
  const colors = {
    "Critical": "red",
    "High": "orange",
    "Medium": "yellow",
    "Low": "gray"
  };
  return colors[priority] || "default";
}
```

## Usage Example

```javascript
// User: "How am I doing on my Q1 goals?"
// Claude analyzes and generates report

const goalId = "abc123..."; // Retrieved from query
const analysis = await analyzeGoalProgress(goalId);

console.log(`Goal: ${analysis.goalName}`);
console.log(`Status: ${analysis.status.emoji} ${analysis.status.severity}`);
console.log(`Progress: ${analysis.progress.current}% (Expected: ${analysis.progress.expected}%)`);
console.log(`Days Remaining: ${analysis.timeline.daysRemaining}`);

// Create visual report
const report = await createProgressReportPage(goalId, analysis);
console.log(`Report created: https://notion.so/${report.id.replace(/-/g, '')}`);
```

## Batch Analysis

```javascript
async function analyzeAllActiveGoals() {
  const activeGoals = await notion.databases.query({
    database_id: goalsDbId,
    filter: {
      property: "Status",
      select: { equals: "In Progress" }
    }
  });
  
  const analyses = [];
  
  for (const goal of activeGoals.results) {
    const analysis = await analyzeGoalProgress(goal.id);
    analyses.push(analysis);
  }
  
  // Group by status
  const grouped = {
    ahead: analyses.filter(a => a.status.severity === "ahead"),
    onTrack: analyses.filter(a => a.status.severity === "on-track"),
    atRisk: analyses.filter(a => a.status.severity === "at-risk"),
    critical: analyses.filter(a => a.status.severity === "critical")
  };
  
  return grouped;
}
```

## Automated Recommendations

The analysis automatically suggests:
- Timeline adjustments
- Task breakdown strategies
- Priority realignments
- Resource allocation
- Celebration moments
