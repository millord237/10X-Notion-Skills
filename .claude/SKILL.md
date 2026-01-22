---
name: notion-comprehensive-system
description: Create and manage complete productivity and wellness systems in Notion including goal tracking with progress analytics, fitness logging with workout programs, nutrition planning with macro tracking and meal plans, daily check-ins with reflection prompts, habit tracking, and cross-database analytics. Use when user mentions setting up Notion workspaces, tracking goals, logging workouts or meals, creating daily planning systems, building databases, or analyzing productivity/health data. Requires Notion MCP server to be configured.
allowed-tools: Read, Write, Bash
---

# Notion Comprehensive System

Build sophisticated, interconnected productivity and wellness systems in Notion with automatic progress tracking, smart analytics, and cross-database insights.

## Quick Start

When you ask me to set up your Notion system, I will:

1. **Create Core Databases** - Goals, Tasks, Projects, Daily Check-ins
2. **Set Up Fitness Tracking** - Workouts, Exercises, Body Measurements  
3. **Build Nutrition System** - Meals, Recipes, Ingredients, Grocery Lists
4. **Configure Wellness** - Habits, Mood Logs, Sleep Tracking
5. **Connect Everything** - Smart relations linking all systems
6. **Add Formulas** - Automatic progress calculations, macro totals, streaks
7. **Create Views** - Dashboards, calendars, boards, galleries for each database
8. **Generate Templates** - Daily check-in, weekly review, workout log, meal plan templates

## What You Can Do

### Goal Tracking
```
"Create a goal to run a marathon in 6 months"
→ I create the main goal with timeline
→ Generate 24 weekly sub-goals automatically  
→ Set up progress tracking with formulas
→ Create action items and milestones
→ Link to workout program

"How am I doing on my 2025 goals?"
→ I analyze all active goals
→ Calculate actual vs expected progress
→ Identify at-risk goals
→ Provide specific recommendations
→ Generate visual progress report
```

### Daily Planning
```
"Create my morning check-in"
→ I create today's planning page
→ Show uncompleted tasks from yesterday
→ Display today's schedule
→ Prompt for top 3 priorities
→ Reference active goals
→ Add evening reflection sections

"What should I focus on today?"
→ I analyze your goals and deadlines
→ Show high-priority tasks
→ Suggest time blocks
→ Identify urgent items
```

### Fitness Tracking
```
"Log workout: bench press 3x8 at 185lbs, squats 3x10 at 225lbs, deadlift 1x5 at 315lbs"
→ I create workout entry
→ Log each exercise with sets/reps/weight
→ Calculate total volume
→ Update personal records
→ Link to active fitness goals

"Show my strength progress over the last month"
→ I query workout history
→ Calculate volume trends
→ Show personal record improvements
→ Generate progress visualization
→ Identify weak points
```

### Nutrition Planning
```
"Generate a meal plan for next week with 2000 calories and 150g protein per day"
→ I query your recipe database
→ Generate balanced 7-day plan
→ Calculate daily macros (protein, carbs, fats)
→ Create meal entries for each day
→ Generate shopping list with ingredients
→ Provide meal prep timeline

"Add a recipe: Chicken Stir-Fry - 400 calories, 35g protein, 30g carbs, 15g fat"
→ I create recipe entry
→ Add to recipe database
→ Link ingredients
→ Calculate per-serving macros
→ Make available for future meal plans
```

### Analytics & Insights
```
"Show correlation between my sleep and productivity"
→ I query check-in database for sleep hours
→ Query tasks database for completion rates
→ Calculate correlation coefficient
→ Generate visualization
→ Provide insights and recommendations

"Generate my Q1 progress report"
→ I analyze all Q1 goals
→ Calculate completion rates
→ Show workout frequency
→ Summarize nutrition adherence
→ Create comprehensive report page
→ Identify patterns and trends
```

## Database Schemas

I create these interconnected databases:

### Goals Database
```
Properties:
- Name (title)
- Status (select: Not Started, In Progress, On Hold, Completed, Abandoned)
- Goal Type (select: Annual, Quarterly, Monthly, Weekly, Daily)
- Progress (number: 0-100%)
- Start Date (date)
- Target Date (date)
- Category (multi-select: Career, Health, Finance, Personal, Relationships, Learning)
- Priority (select: Critical, High, Medium, Low)
- Parent Goal (relation → Goals)
- Sub-goals (relation → Goals)
- Action Items (relation → Tasks)
- Related Workouts (relation → Workouts)
- Related Check-ins (relation → Daily Check-ins)
- Notes (rich text)

Formulas:
- Days Remaining: dateBetween(prop("Target Date"), now(), "days")
- Time Elapsed %: (dateBetween(now(), prop("Start Date"), "days") / dateBetween(prop("Target Date"), prop("Start Date"), "days")) * 100
- Expected Progress: Time Elapsed %
- Progress Variance: Progress - Expected Progress
- On Track: if(Progress >= Expected Progress, "✅", "⚠️")

Views:
- Dashboard (Gallery, grouped by Status)
- Timeline (Timeline, by Target Date)
- By Category (Board, grouped by Category)
- At Risk (Table, filtered: Progress < Expected Progress - 10)
- Review Queue (Table, sorted by Last Reviewed)
```

### Tasks Database
```
Properties:
- Task Name (title)
- Status (select: To Do, In Progress, Done, Blocked)
- Priority (select: Critical, High, Medium, Low)
- Date (date)
- Time Block (select: Morning, Midday, Afternoon, Evening, Anytime)
- Estimated Duration (number, in minutes)
- Actual Duration (number, in minutes)
- Energy Level Required (select: High, Medium, Low)
- Related Goal (relation → Goals)
- Related Project (relation → Projects)
- Tags (multi-select)

Views:
- Today (Calendar + List, filtered by today)
- This Week (Board, grouped by day)
- By Time Block (Board, grouped by Time Block)
- Backlog (Table, no date set)
- Completed (Table, Status = Done, sorted by Date desc)
```

### Workouts Database
```
Properties:
- Workout Name (title)
- Date (date)
- Type (select: Cardio, Strength, Flexibility, Sports, Other)
- Duration (number, minutes)
- Intensity (select: 1-10)
- Muscle Groups (multi-select: Chest, Back, Legs, Shoulders, Arms, Core, Full Body)
- Calories Burned (number)
- Exercises (relation → Exercises)
- Total Volume (rollup from Exercises: sum of volume)
- Mood Rating (select: Excellent, Good, Neutral, Tired, Exhausted)
- Related Goal (relation → Goals)
- Notes (rich text)

Views:
- Calendar (Calendar view by Date)
- By Type (Board, grouped by Type)
- By Muscle Group (Board, grouped by Muscle Groups)
- This Week (List, filtered by current week)
- Volume Tracker (Table with Volume calculations)
```

### Exercises Database
```
Properties:
- Exercise Name (title)
- Type (select: Cardio, Strength, Flexibility)
- Primary Muscle (select: Chest, Back, Legs, Shoulders, Arms, Core)
- Equipment (multi-select: Barbell, Dumbbell, Machine, Cable, Bodyweight, Bands)
- Sets (number)
- Reps (number)
- Weight (number)
- Volume (formula: Sets × Reps × Weight)
- Personal Best Weight (number)
- Personal Best Date (date)
- Rest Time (number, seconds)
- Form Notes (rich text)
- Video URL (url)

Views:
- Exercise Library (Gallery)
- By Muscle (Board, grouped by Primary Muscle)
- Personal Records (Table, sorted by Personal Best Weight desc)
- Recently Used (Table, sorted by last used date)
```

### Meals Database
```
Properties:
- Meal Name (title)
- Date (date)
- Meal Type (select: Breakfast, Lunch, Dinner, Snack)
- Recipes (relation → Recipes)
- Total Calories (rollup from Recipes: sum Calories)
- Total Protein (rollup from Recipes: sum Protein)
- Total Carbs (rollup from Recipes: sum Carbs)
- Total Fats (rollup from Recipes: sum Fats)
- Rating (select: 😍 😊 😐 😕 😫)
- Notes (rich text)

Formulas:
- Macro Balance: Shows P/C/F ratio
- Meets Target: Compares to daily goals

Views:
- Meal Calendar (Calendar by Date)
- This Week (List, current week)
- By Type (Board, grouped by Meal Type)
- Macro Dashboard (Table with totals and formulas)
- Favorite Meals (Gallery, filtered by Rating = 😍)
```

### Recipes Database
```
Properties:
- Recipe Name (title)
- Cuisine Type (select: American, Italian, Mexican, Asian, Mediterranean, Other)
- Meal Type (select: Breakfast, Lunch, Dinner, Snack)
- Servings (number)
- Prep Time (number, minutes)
- Cook Time (number, minutes)
- Total Time (formula: Prep Time + Cook Time)
- Calories per Serving (number)
- Protein (number, grams)
- Carbs (number, grams)
- Fats (number, grams)
- Ingredients (relation → Ingredients)
- Instructions (rich text)
- Tags (multi-select: High Protein, Low Carb, Vegetarian, Vegan, Quick, Budget)
- Cost per Serving (number, currency)
- Photos (files)

Views:
- Recipe Gallery (Gallery with photos)
- By Cuisine (Board, grouped by Cuisine Type)
- Quick Meals (List, filtered: Total Time < 30)
- High Protein (List, filtered: Protein > 30g)
- Budget Friendly (Table, sorted by Cost per Serving)
```

### Daily Check-ins Database
```
Properties:
- Date (date, unique)
- Morning Energy (select: High, Medium, Low)
- Top 3 Tasks (rich text)
- Evening Mood (select: Excellent, Good, Neutral, Low, Poor)
- What Went Well (rich text)
- Challenges Faced (rich text)
- Gratitude (rich text)
- Tomorrow's Focus (rich text)
- Workout Completed (checkbox)
- Nutrition Goal Met (checkbox)
- Sleep Hours (number)
- Sleep Quality (select: Excellent, Good, Fair, Poor)
- Overall Rating (select: 1-10)
- Habits Completed (relation → Habits)
- Tasks Completed (rollup from Tasks: count Done)
- Related Goals (relation → Goals)

Formulas:
- Productivity Score: (Tasks Completed / 10) * Overall Rating
- Wellness Score: (Sleep Hours / 8) * (if Workout Completed, 1.2, 1) * (if Nutrition Goal Met, 1.2, 1)

Views:
- Today (Full page view)
- This Week (Table, current week)
- Monthly Trends (Gallery by week)
- Streak Tracker (Table with consecutive days formula)
- Insights (Table with averages and correlations)
```

### Habits Database
```
Properties:
- Habit Name (title)
- Category (select: Health, Productivity, Learning, Social, Self-Care)
- Frequency Goal (select: Daily, Weekly, Monthly)
- Current Streak (number)
- Longest Streak (number)
- Last Completed (date)
- Related Goal (relation → Goals)
- Notes (rich text)

Views:
- Active Habits (Board, grouped by Category)
- Streak Leaderboard (Table, sorted by Current Streak desc)
- Due Today (List, filtered by Frequency Goal)
```

## Implementation Examples

### Creating a Goal with Sub-goals

When you say: "Create a goal to lose 20 pounds in 4 months"

I will:

```javascript
// 1. Create main goal
const mainGoal = await notion.pages.create({
  parent: { database_id: goalsDbId },
  properties: {
    "Name": { title: [{ text: { content: "Lose 20 pounds" }}] },
    "Goal Type": { select: { name: "Quarterly" }},
    "Start Date": { date: { start: today }},
    "Target Date": { date: { start: fourMonthsFromNow }},
    "Category": { multi_select: [{ name: "Health" }]},
    "Priority": { select: { name: "High" }},
    "Progress": { number: 0 }
  }
});

// 2. Generate weekly sub-goals (16 weeks)
const weeksInGoal = 16;
const poundsPerWeek = 20 / 16; // ~1.25 lbs/week

for (let week = 1; week <= weeksInGoal; week++) {
  await notion.pages.create({
    parent: { database_id: goalsDbId },
    properties: {
      "Name": { title: [{ text: { content: `Week ${week}: Lose ${(poundsPerWeek * week).toFixed(1)} lbs total` }}] },
      "Goal Type": { select: { name: "Weekly" }},
      "Parent Goal": { relation: [{ id: mainGoal.id }]},
      "Progress": { number: 0 }
    }
  });
}

// 3. Create supporting workout program
await notion.pages.create({
  parent: { database_id: programsDbId },
  properties: {
    "Name": { title: [{ text: { content: "4-Month Weight Loss Program" }}] },
    "Related Goal": { relation: [{ id: mainGoal.id }]}
  }
});

// 4. Create action items
const actions = [
  "Set up meal tracking system",
  "Schedule 4 workouts per week",
  "Calculate daily calorie target",
  "Take starting measurements and photos",
  "Buy meal prep containers"
];

for (const action of actions) {
  await notion.pages.create({
    parent: { database_id: tasksDbId },
    properties: {
      "Task Name": { title: [{ text: { content: action }}] },
      "Related Goal": { relation: [{ id: mainGoal.id }]},
      "Priority": { select: { name: "High" }}
    }
  });
}
```

### Daily Check-in Creation

When you say: "Create my morning check-in"

I will:

```javascript
// 1. Check if today's check-in exists
const today = new Date().toISOString().split('T')[0];
const existing = await notion.databases.query({
  database_id: checkInsDbId,
  filter: { property: "Date", date: { equals: today }}
});

if (existing.results.length > 0) {
  return "You already have a check-in for today!";
}

// 2. Get yesterday's uncompleted tasks
const yesterday = new Date(Date.now() - 86400000).toISOString().split('T')[0];
const incompleteTasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: {
    and: [
      { property: "Date", date: { equals: yesterday }},
      { property: "Status", select: { does_not_equal: "Done" }}
    ]
  }
});

// 3. Get today's scheduled tasks
const todayTasks = await notion.databases.query({
  database_id: tasksDbId,
  filter: { property: "Date", date: { equals: today }},
  sorts: [{ property: "Time Block", direction: "ascending" }]
});

// 4. Get active goals
const activeGoals = await notion.databases.query({
  database_id: goalsDbId,
  filter: { property: "Status", select: { equals: "In Progress" }},
  sorts: [{ property: "Priority", direction: "ascending" }],
  page_size: 5
});

// 5. Create check-in page with all context
const checkIn = await notion.pages.create({
  parent: { database_id: checkInsDbId },
  properties: {
    "Date": { date: { start: today }}
  },
  children: [
    {
      type: "heading_1",
      heading_1: {
        rich_text: [{ text: { content: "🌅 Morning Check-In" }}],
        color: "blue_background"
      }
    },
    {
      type: "callout",
      callout: {
        rich_text: [{ text: { content: `Good morning! Today is ${new Date().toLocaleDateString('en-US', { weekday: 'long', month: 'long', day: 'numeric' })}` }}],
        icon: { emoji: "☀️" }
      }
    },
    {
      type: "heading_2",
      heading_2: { rich_text: [{ text: { content: "📋 Unfinished from Yesterday" }}] }
    },
    ...incompleteTasks.results.map(task => ({
      type: "to_do",
      to_do: {
        rich_text: [{ text: { content: task.properties["Task Name"].title[0].plain_text }}],
        checked: false
      }
    })),
    {
      type: "heading_2",
      heading_2: { rich_text: [{ text: { content: "📅 Today's Schedule" }}] }
    },
    ...todayTasks.results.map(task => ({
      type: "bulleted_list_item",
      bulleted_list_item: {
        rich_text: [{ text: {
          content: `[${task.properties["Time Block"]?.select?.name || "Anytime"}] ${task.properties["Task Name"].title[0].plain_text}`
        }}]
      }
    })),
    {
      type: "heading_2",
      heading_2: { rich_text: [{ text: { content: "🎯 Active Goals" }}] }
    },
    ...activeGoals.results.map(goal => ({
      type: "bulleted_list_item",
      bulleted_list_item: {
        rich_text: [{ text: {
          content: `${goal.properties.Name.title[0].plain_text} - ${goal.properties.Progress.number}% complete`
        }}]
      }
    })),
    {
      type: "divider"
    },
    {
      type: "heading_2",
      heading_2: {
        rich_text: [{ text: { content: "⭐ Today's Top 3 Priorities" }}],
        color: "orange_background"
      }
    },
    { type: "numbered_list_item", numbered_list_item: { rich_text: [{ text: { content: "" }}] }},
    { type: "numbered_list_item", numbered_list_item: { rich_text: [{ text: { content: "" }}] }},
    { type: "numbered_list_item", numbered_list_item: { rich_text: [{ text: { content: "" }}] }},
    {
      type: "divider"
    },
    {
      type: "heading_1",
      heading_1: {
        rich_text: [{ text: { content: "🌙 Evening Reflection" }}],
        color: "purple_background"
      }
    },
    {
      type: "callout",
      callout: {
        rich_text: [{ text: { content: "Complete this section at the end of your day." }}],
        icon: { emoji: "🌙" }
      }
    },
    {
      type: "heading_2",
      heading_2: { rich_text: [{ text: { content: "✨ What Went Well?" }}] }
    },
    { type: "paragraph", paragraph: { rich_text: [] }},
    {
      type: "heading_2",
      heading_2: { rich_text: [{ text: { content: "🔧 What Could Be Improved?" }}] }
    },
    { type: "paragraph", paragraph: { rich_text: [] }},
    {
      type: "heading_2",
      heading_2: { rich_text: [{ text: { content: "🙏 Three Things I'm Grateful For" }}] }
    },
    { type: "numbered_list_item", numbered_list_item: { rich_text: [] }},
    { type: "numbered_list_item", numbered_list_item: { rich_text: [] }},
    { type: "numbered_list_item", numbered_list_item: { rich_text: [] }}
  ]
});

return `Created your daily check-in! View at: https://notion.so/${checkIn.id.replace(/-/g, '')}`;
```

### Goal Progress Analysis

When you say: "How am I doing on my 2025 goals?"

I will:

```javascript
// 1. Query all 2025 goals
const goals2025 = await notion.databases.query({
  database_id: goalsDbId,
  filter: {
    and: [
      { property: "Start Date", date: { on_or_after: "2025-01-01" }},
      { property: "Status", select: { is_not_empty: true }}
    ]
  }
});

// 2. Analyze each goal
const analyses = [];

for (const goal of goals2025.results) {
  const startDate = new Date(goal.properties["Start Date"].date.start);
  const targetDate = new Date(goal.properties["Target Date"].date.start);
  const today = new Date();
  
  const totalDays = (targetDate - startDate) / (1000 * 60 * 60 * 24);
  const elapsedDays = (today - startDate) / (1000 * 60 * 60 * 24);
  const remainingDays = (targetDate - today) / (1000 * 60 * 60 * 24);
  
  const expectedProgress = (elapsedDays / totalDays) * 100;
  const actualProgress = goal.properties.Progress.number || 0;
  const variance = actualProgress - expectedProgress;
  
  let status;
  if (variance >= 10) status = { level: "ahead", emoji: "🚀", color: "green" };
  else if (variance >= -5) status = { level: "on-track", emoji: "✅", color: "blue" };
  else if (variance >= -15) status = { level: "at-risk", emoji: "⚠️", color: "yellow" };
  else status = { level: "critical", emoji: "🔴", color: "red" };
  
  analyses.push({
    name: goal.properties.Name.title[0].plain_text,
    actualProgress: Math.round(actualProgress),
    expectedProgress: Math.round(expectedProgress),
    variance: Math.round(variance),
    daysRemaining: Math.round(remainingDays),
    status: status
  });
}

// 3. Create progress report page
const reportPage = await notion.pages.create({
  parent: { page_id: workspaceId },
  properties: {
    title: { title: [{ text: { content: "2025 Goals Progress Report" }}] }
  },
  children: [
    {
      type: "heading_1",
      heading_1: { rich_text: [{ text: { content: "📊 2025 Goals Progress Report" }}] }
    },
    {
      type: "callout",
      callout: {
        rich_text: [{ text: { content: `Generated on ${new Date().toLocaleDateString()}` }}],
        icon: { emoji: "📈" }
      }
    },
    ...analyses.map(a => ({
      type: "toggle",
      toggle: {
        rich_text: [{
          text: {
            content: `${a.status.emoji} ${a.name} - ${a.actualProgress}% (${a.status.level})`
          }
        }],
        color: `${a.status.color}_background`,
        children: [
          {
            type: "paragraph",
            paragraph: {
              rich_text: [{
                text: {
                  content: `Progress: ${a.actualProgress}% (Expected: ${a.expectedProgress}%)\nVariance: ${a.variance > 0 ? '+' : ''}${a.variance}%\nDays Remaining: ${a.daysRemaining}`
                }
              }]
            }
          }
        ]
      }
    }))
  ]
});

// 4. Generate summary statistics
const summary = {
  total: analyses.length,
  ahead: analyses.filter(a => a.status.level === "ahead").length,
  onTrack: analyses.filter(a => a.status.level === "on-track").length,
  atRisk: analyses.filter(a => a.status.level === "at-risk").length,
  critical: analyses.filter(a => a.status.level === "critical").length
};

return `
2025 Goals Summary:
Total: ${summary.total} goals
🚀 Ahead: ${summary.ahead}
✅ On Track: ${summary.onTrack}
⚠️ At Risk: ${summary.atRisk}
🔴 Critical: ${summary.critical}

Full report: https://notion.so/${reportPage.id.replace(/-/g, '')}
`;
```

### Workout Logging

When you say: "Log workout: bench press 3x8 at 185lbs, squats 3x10 at 225lbs"

I will:

```javascript
// 1. Create workout entry
const workout = await notion.pages.create({
  parent: { database_id: workoutsDbId },
  properties: {
    "Workout Name": { title: [{ text: { content: "Strength Training" }}] },
    "Date": { date: { start: new Date().toISOString().split('T')[0] }},
    "Type": { select: { name: "Strength" }},
    "Muscle Groups": { multi_select: [{ name: "Chest" }, { name: "Legs" }]}
  }
});

// 2. Log bench press
const benchPress = await notion.pages.create({
  parent: { database_id: exercisesDbId },
  properties: {
    "Exercise Name": { title: [{ text: { content: "Bench Press" }}] },
    "Type": { select: { name: "Strength" }},
    "Primary Muscle": { select: { name: "Chest" }},
    "Sets": { number: 3 },
    "Reps": { number: 8 },
    "Weight": { number: 185 }
  }
});

// 3. Log squats
const squats = await notion.pages.create({
  parent: { database_id: exercisesDbId },
  properties: {
    "Exercise Name": { title: [{ text: { content: "Squats" }}] },
    "Type": { select: { name: "Strength" }},
    "Primary Muscle": { select: { name: "Legs" }},
    "Sets": { number: 3 },
    "Reps": { number: 10 },
    "Weight": { number: 225 }
  }
});

// 4. Link exercises to workout
await notion.pages.update({
  page_id: workout.id,
  properties: {
    "Exercises": {
      relation: [
        { id: benchPress.id },
        { id: squats.id }
      ]
    }
  }
});

// 5. Check for personal records
const existingBenchPR = await notion.databases.query({
  database_id: exercisesDbId,
  filter: {
    and: [
      { property: "Exercise Name", title: { equals: "Bench Press" }},
      { property: "Weight", number: { greater_than: 185 }}
    ]
  }
});

if (existingBenchPR.results.length === 0) {
  // New PR!
  await notion.pages.update({
    page_id: benchPress.id,
    properties: {
      "Personal Best Weight": { number: 185 },
      "Personal Best Date": { date: { start: new Date().toISOString().split('T')[0] }}
    }
  });
  
  return "Workout logged! 🎉 NEW PERSONAL RECORD on Bench Press: 185lbs!";
}

return "Workout logged successfully!";
```

### Meal Plan Generation

When you say: "Generate a meal plan for next week with 2000 calories and 150g protein"

I will:

```javascript
// 1. Query available recipes
const recipes = await notion.databases.query({
  database_id: recipesDbId,
  filter: {
    property: "Calories per Serving",
    number: { less_than_or_equal_to: 700 }
  }
});

// 2. Categorize recipes by meal type
const recipesByType = {
  Breakfast: recipes.results.filter(r => r.properties["Meal Type"].select?.name === "Breakfast"),
  Lunch: recipes.results.filter(r => r.properties["Meal Type"].select?.name === "Lunch"),
  Dinner: recipes.results.filter(r => r.properties["Meal Type"].select?.name === "Dinner"),
  Snack: recipes.results.filter(r => r.properties["Meal Type"].select?.name === "Snack")
};

// 3. Generate 7-day plan
const weekPlan = [];
const startDate = new Date();
startDate.setDate(startDate.getDate() + 1); // Start tomorrow

for (let day = 0; day < 7; day++) {
  const date = new Date(startDate);
  date.setDate(date.getDate() + day);
  const dateStr = date.toISOString().split('T')[0];
  
  const dayPlan = {
    date: dateStr,
    meals: [],
    totals: { calories: 0, protein: 0, carbs: 0, fats: 0 }
  };
  
  // Select breakfast (target: ~500 cal)
  const breakfast = recipesByType.Breakfast[Math.floor(Math.random() * recipesByType.Breakfast.length)];
  dayPlan.meals.push({ type: "Breakfast", recipe: breakfast });
  
  // Select lunch (target: ~600 cal)
  const lunch = recipesByType.Lunch[Math.floor(Math.random() * recipesByType.Lunch.length)];
  dayPlan.meals.push({ type: "Lunch", recipe: lunch });
  
  // Select dinner (target: ~700 cal)
  const dinner = recipesByType.Dinner[Math.floor(Math.random() * recipesByType.Dinner.length)];
  dayPlan.meals.push({ type: "Dinner", recipe: dinner });
  
  // Calculate totals
  for (const meal of dayPlan.meals) {
    dayPlan.totals.calories += meal.recipe.properties["Calories per Serving"].number;
    dayPlan.totals.protein += meal.recipe.properties.Protein.number;
    dayPlan.totals.carbs += meal.recipe.properties.Carbs.number;
    dayPlan.totals.fats += meal.recipe.properties.Fats.number;
  }
  
  // Add snack if under calorie target
  if (dayPlan.totals.calories < 1800 && recipesByType.Snack.length > 0) {
    const snack = recipesByType.Snack[Math.floor(Math.random() * recipesByType.Snack.length)];
    dayPlan.meals.push({ type: "Snack", recipe: snack });
    dayPlan.totals.calories += snack.properties["Calories per Serving"].number;
    dayPlan.totals.protein += snack.properties.Protein.number;
  }
  
  weekPlan.push(dayPlan);
}

// 4. Create meal entries in database
for (const day of weekPlan) {
  for (const meal of day.meals) {
    await notion.pages.create({
      parent: { database_id: mealsDbId },
      properties: {
        "Meal Name": {
          title: [{ text: { content: meal.recipe.properties["Recipe Name"].title[0].plain_text }}]
        },
        "Date": { date: { start: day.date }},
        "Meal Type": { select: { name: meal.type }},
        "Recipes": { relation: [{ id: meal.recipe.id }] }
      }
    });
  }
}

// 5. Generate shopping list
const allIngredients = [];
for (const day of weekPlan) {
  for (const meal of day.meals) {
    const recipeIngredients = meal.recipe.properties.Ingredients?.relation || [];
    allIngredients.push(...recipeIngredients);
  }
}

// Deduplicate and create grocery list
const uniqueIngredients = [...new Set(allIngredients.map(i => i.id))];

await notion.pages.create({
  parent: { database_id: groceryListDbId },
  properties: {
    "List Name": { title: [{ text: { content: `Grocery List - Week of ${weekPlan[0].date}` }}] },
    "Ingredients": { relation: uniqueIngredients.map(id => ({ id })) }
  }
});

// 6. Generate summary
const avgCals = weekPlan.reduce((sum, d) => sum + d.totals.calories, 0) / 7;
const avgProtein = weekPlan.reduce((sum, d) => sum + d.totals.protein, 0) / 7;

return `
Meal plan created for next week!

Average per day:
- ${Math.round(avgCals)} calories (target: 2000)
- ${Math.round(avgProtein)}g protein (target: 150g)

Total meals planned: ${weekPlan.length * 3} meals + snacks
Grocery list created with ${uniqueIngredients.length} unique ingredients
`;
```

## Advanced Features

### Cross-Database Analytics

I can analyze patterns across your systems:

```
"Show correlation between sleep and productivity"
→ I query check-ins for sleep hours and task completion
→ Calculate correlation coefficient
→ Generate scatter plot data
→ Provide insights on optimal sleep

"What workout days lead to best productivity?"
→ I compare workout dates with task completion rates
→ Identify patterns
→ Suggest optimal workout timing
```

### Cascading Templates

When you create items, I automatically generate related content:

- Create goal → Auto-generate weekly sub-goals
- Create workout program → Auto-create exercise templates
- Create meal plan → Auto-generate shopping list
- Complete check-in → Update habit streaks

### Smart Suggestions

I provide intelligent recommendations:

```
"What should I work on today?"
→ Analyzes goals, deadlines, energy levels
→ Suggests optimal tasks for current time block
→ Considers your typical productivity patterns

"Suggest a workout"
→ Checks last workout date
→ Identifies muscle groups needing attention
→ Recommends appropriate intensity
→ Links to relevant exercises
```

## Reference Files

For complete implementation details, see:
- [reference/database-schemas.md](reference/database-schemas.md) - Full property definitions
- [reference/formulas.md](reference/formulas.md) - All Notion formulas
- [reference/api-patterns.md](reference/api-patterns.md) - Common API usage patterns
- [examples/complete-workflows.md](examples/complete-workflows.md) - Full workflow implementations

## Troubleshooting

**"Cannot access Notion"**
→ Verify Notion MCP server is configured in `claude_desktop_config.json`
→ Check your integration token is valid
→ Ensure integration is connected to your workspace pages

**"Database not found"**
→ Make sure I've created the databases first
→ Check integration has access to parent page
→ Verify database IDs are correct

**"Relation not working"**
→ Both databases must exist before creating relations
→ Relation property must reference correct database

**"Formula shows error"**
→ Formulas need proper null checks
→ I handle this automatically in my implementations

## Version

Skill Version: 1.0.0
Last Updated: 2025-01-12
Compatible with: Notion MCP Server 1.0+

## Notes

- All database IDs are stored and referenced automatically
- I preserve existing data when updating schemas
- Multiple views are created for each database
- Formulas calculate automatically
- Relations update in real-time
- Templates can be customized after creation

This skill requires an active Notion MCP server connection. All operations use the official Notion API via the MCP protocol.
