# Daily Check-In Flow

Complete implementation for creating and managing daily check-ins in Notion.

## Overview

This flow creates a structured daily check-in page that includes:
- Morning planning section with yesterday's uncompleted tasks
- Today's schedule preview
- Priority setting prompts
- Evening reflection sections
- Habit tracking links

## Implementation

### Step 1: Check for Existing Check-In

```javascript
async function createDailyCheckIn() {
  const today = new Date().toISOString().split('T')[0];
  
  // Query for existing check-in
  const existingCheckIns = await notion.databases.query({
    database_id: checkInsDbId,
    filter: {
      property: "Date",
      date: {
        equals: today
      }
    }
  });
  
  if (existingCheckIns.results.length > 0) {
    return {
      status: "exists",
      message: "You already have a check-in for today!",
      pageId: existingCheckIns.results[0].id,
      url: `https://notion.so/${existingCheckIns.results[0].id.replace(/-/g, '')}`
    };
  }
  
  // Continue to Step 2...
}
```

### Step 2: Query Yesterday's Tasks

```javascript
// Get uncompleted tasks from yesterday
const yesterday = new Date(Date.now() - 86400000).toISOString().split('T')[0];

const uncompletedTasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      {
        property: "Date",
        date: { equals: yesterday }
      },
      {
        property: "Status",
        select: { does_not_equal: "Done" }
      }
    ]
  },
  sorts: [
    {
      property: "Priority",
      direction: "ascending"
    }
  ]
});
```

### Step 3: Query Today's Schedule

```javascript
// Get today's scheduled tasks
const todaysTasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    property: "Date",
    date: { equals: today }
  },
  sorts: [
    {
      property: "Time Block",
      direction: "ascending"
    },
    {
      property: "Priority",
      direction: "ascending"
    }
  ]
});
```

### Step 4: Query Active Goals

```javascript
// Get active goals for context
const activeGoals = await notion.databases.query({
  database_id: goalsDbId,
  filter: {
    property: "Status",
    select: { equals: "In Progress" }
  },
  sorts: [
    {
      property: "Priority",
      direction: "ascending"
    }
  ],
  page_size: 5
});
```

### Step 5: Create Check-In Page

```javascript
const checkInPage = await notion.pages.create({
  parent: {
    database_id: checkInsDbId
  },
  properties: {
    "Date": {
      date: { start: today }
    }
  },
  children: [
    // Morning Section Header
    {
      object: "block",
      type: "heading_1",
      heading_1: {
        rich_text: [{
          type: "text",
          text: { content: "🌅 Morning Check-In" }
        }],
        color: "blue_background"
      }
    },
    
    // Morning Energy Callout
    {
      object: "block",
      type: "callout",
      callout: {
        rich_text: [{
          type: "text",
          text: { content: "How's your energy this morning? Set this in the page properties above." }
        }],
        icon: { emoji: "⚡" },
        color: "yellow_background"
      }
    },
    
    // Yesterday's Unfinished Tasks
    {
      object: "block",
      type: "heading_2",
      heading_2: {
        rich_text: [{
          type: "text",
          text: { content: "📋 Yesterday's Unfinished Tasks" }
        }]
      }
    },
    
    // Add unfinished tasks as to-dos
    ...uncompletedTasks.results.map(task => ({
      object: "block",
      type: "to_do",
      to_do: {
        rich_text: [{
          type: "text",
          text: {
            content: task.properties["Task Name"].title[0].plain_text
          }
        }],
        checked: false,
        color: getPriorityColor(task.properties.Priority.select?.name)
      }
    })),
    
    // Today's Schedule
    {
      object: "block",
      type: "heading_2",
      heading_2: {
        rich_text: [{
          type: "text",
          text: { content: "📅 Today's Schedule" }
        }]
      }
    },
    
    // Group tasks by time block
    ...groupByTimeBlock(todaysTasks.results),
    
    // Active Goals Reference
    {
      object: "block",
      type: "heading_2",
      heading_2: {
        rich_text: [{
          type: "text",
          text: { content: "🎯 Active Goals to Consider" }
        }]
      }
    },
    
    ...activeGoals.results.map(goal => ({
      object: "block",
      type: "bulleted_list_item",
      bulleted_list_item: {
        rich_text: [{
          type: "text",
          text: {
            content: `${goal.properties.Name.title[0].plain_text} - ${goal.properties.Progress.number}% complete`
          }
        }]
      }
    })),
    
    // Top 3 Priorities Section
    {
      object: "block",
      type: "divider",
      divider: {}
    },
    
    {
      object: "block",
      type: "heading_2",
      heading_2: {
        rich_text: [{
          type: "text",
          text: { content: "⭐ Today's Top 3 Priorities" }
        }],
        color: "orange_background"
      }
    },
    
    {
      object: "block",
      type: "callout",
      callout: {
        rich_text: [{
          type: "text",
          text: { content: "What are the 3 most important things to accomplish today?" }
        }],
        icon: { emoji: "💡" }
      }
    },
    
    { object: "block", type: "numbered_list_item", numbered_list_item: { rich_text: [{ text: { content: "" }}] }},
    { object: "block", type: "numbered_list_item", numbered_list_item: { rich_text: [{ text: { content: "" }}] }},
    { object: "block", type: "numbered_list_item", numbered_list_item: { rich_text: [{ text: { content: "" }}] }},
    
    // Evening Reflection Section
    {
      object: "block",
      type: "divider",
      divider: {}
    },
    
    {
      object: "block",
      type: "heading_1",
      heading_1: {
        rich_text: [{
          type: "text",
          text: { content: "🌙 Evening Reflection" }
        }],
        color: "purple_background"
      }
    },
    
    {
      object: "block",
      type: "callout",
      callout: {
        rich_text: [{
          type: "text",
          text: { content: "Complete this section at the end of your day. Take a moment to reflect on what happened." }
        }],
        icon: { emoji: "🌙" }
      }
    },
    
    {
      object: "block",
      type: "heading_2",
      heading_2: {
        rich_text: [{
          type: "text",
          text: { content: "✨ What Went Well Today?" }
        }]
      }
    },
    
    { object: "block", type: "paragraph", paragraph: { rich_text: [] }},
    
    {
      object: "block",
      type: "heading_2",
      heading_2: {
        rich_text: [{
          type: "text",
          text: { content: "🔧 What Could Be Improved?" }
        }]
      }
    },
    
    { object: "block", type: "paragraph", paragraph: { rich_text: [] }},
    
    {
      object: "block",
      type: "heading_2",
      heading_2: {
        rich_text: [{
          type: "text",
          text: { content: "🙏 Three Things I'm Grateful For" }
        }]
      }
    },
    
    { object: "block", type: "numbered_list_item", numbered_list_item: { rich_text: [{ text: { content: "" }}] }},
    { object: "block", type: "numbered_list_item", numbered_list_item: { rich_text: [{ text: { content: "" }}] }},
    { object: "block", type: "numbered_list_item", numbered_list_item: { rich_text: [{ text: { content: "" }}] }},
    
    {
      object: "block",
      type: "heading_2",
      heading_2: {
        rich_text: [{
          type: "text",
          text: { content: "🔮 Tomorrow's Focus" }
        }]
      }
    },
    
    { object: "block", type: "paragraph", paragraph: { rich_text: [] }},
    
    // Habit & Metrics Reminder
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
          type: "text",
          text: { content: "Don't forget to update: Sleep hours, Workout checkbox, Nutrition checkbox, and Overall rating in the page properties!" }
        }],
        icon: { emoji: "📊" },
        color: "gray_background"
      }
    }
  ]
});

return {
  status: "created",
  message: "Created your daily check-in!",
  pageId: checkInPage.id,
  url: `https://notion.so/${checkInPage.id.replace(/-/g, '')}`
};
```

## Helper Functions

### Group Tasks by Time Block

```javascript
function groupByTimeBlock(tasks) {
  const timeBlocks = {
    "Morning": [],
    "Midday": [],
    "Afternoon": [],
    "Evening": [],
    "Anytime": []
  };
  
  tasks.forEach(task => {
    const timeBlock = task.properties["Time Block"]?.select?.name || "Anytime";
    timeBlocks[timeBlock].push(task);
  });
  
  const blocks = [];
  
  for (const [blockName, blockTasks] of Object.entries(timeBlocks)) {
    if (blockTasks.length > 0) {
      blocks.push({
        object: "block",
        type: "heading_3",
        heading_3: {
          rich_text: [{
            type: "text",
            text: { content: `${getTimeBlockEmoji(blockName)} ${blockName}` }
          }]
        }
      });
      
      blockTasks.forEach(task => {
        blocks.push({
          object: "block",
          type: "bulleted_list_item",
          bulleted_list_item: {
            rich_text: [{
              type: "text",
              text: {
                content: `${task.properties["Task Name"].title[0].plain_text} [${task.properties.Priority?.select?.name || "Medium"}]`
              }
            }]
          }
        });
      });
    }
  }
  
  return blocks;
}

function getTimeBlockEmoji(blockName) {
  const emojis = {
    "Morning": "🌅",
    "Midday": "☀️",
    "Afternoon": "🌤️",
    "Evening": "🌆",
    "Anytime": "⏰"
  };
  return emojis[blockName] || "⏰";
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
// Claude will automatically call this when user says:
// "Create my morning check-in"
// "Do my daily check-in"
// "Set up today's planning"

const result = await createDailyCheckIn();

console.log(result.message);
console.log(`View at: ${result.url}`);
```

## Customization Options

Users can customize by:
1. Adding custom sections to the template
2. Changing which tasks are shown
3. Modifying the prompts
4. Adding automation for property updates
5. Integrating with other databases

## Next Steps

After creating check-in:
1. Fill in morning energy level
2. Review and check off tasks
3. Set top 3 priorities
4. Return in evening for reflection
5. Update metrics (sleep, workouts, etc.)
