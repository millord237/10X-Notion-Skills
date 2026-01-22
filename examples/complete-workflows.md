# Complete Workflow Examples

This file contains full, working implementations of common workflows using the Notion MCP server.

## Workflow 1: Complete System Setup

This creates all databases with proper relations and formulas.

```javascript
async function setupCompleteSystem(workspacePageId) {
  console.log("Starting complete Notion system setup...");
  
  // Step 1: Create standalone databases (no relations)
  const databases = {};
  
  // Goals database
  databases.goals = await notion.databases.create({
    parent: { page_id: workspacePageId },
    title: [{ text: { content: "Goals" }}],
    icon: { emoji: "🎯" },
    properties: {
      "Name": { title: {} },
      "Status": {
        select: {
          options: [
            { name: "Not Started", color: "gray" },
            { name: "In Progress", color: "blue" },
            { name: "Completed", color: "green" }
          ]
        }
      },
      "Progress": { number: { format: "percent" }},
      "Start Date": { date: {} },
      "Target Date": { date: {} }
    }
  });
  
  // Tasks database
  databases.tasks = await notion.databases.create({
    parent: { page_id: workspacePageId },
    title: [{ text: { content: "Tasks" }}],
    icon: { emoji: "✅" },
    properties: {
      "Task Name": { title: {} },
      "Status": {
        select: {
          options: [
            { name: "To Do", color: "gray" },
            { name: "In Progress", color: "blue" },
            { name: "Done", color: "green" }
          ]
        }
      },
      "Priority": {
        select: {
          options: [
            { name: "Critical", color: "red" },
            { name: "High", color: "orange" },
            { name: "Medium", color: "yellow" },
            { name: "Low", color: "gray" }
          ]
        }
      },
      "Date": { date: {} }
    }
  });
  
  // Step 2: Add relations between databases
  await notion.databases.update({
    database_id: databases.goals.id,
    properties: {
      "Action Items": {
        relation: { database_id: databases.tasks.id }
      }
    }
  });
  
  await notion.databases.update({
    database_id: databases.tasks.id,
    properties: {
      "Related Goal": {
        relation: { database_id: databases.goals.id }
      }
    }
  });
  
  // Step 3: Add formulas
  await notion.databases.update({
    database_id: databases.goals.id,
    properties: {
      "Days Remaining": {
        formula: {
          expression: 'dateBetween(prop("Target Date"), now(), "days")'
        }
      },
      "Expected Progress": {
        formula: {
          expression: '(dateBetween(now(), prop("Start Date"), "days") / dateBetween(prop("Target Date"), prop("Start Date"), "days")) * 100'
        }
      }
    }
  });
  
  console.log("System setup complete!");
  return databases;
}
```

## Workflow 2: Daily Check-In Creation

Full implementation with all context gathering.

```javascript
async function createDailyCheckIn() {
  const today = new Date().toISOString().split('T')[0];
  
  // Check for existing
  const existing = await notion.databases.query({
    database_id: checkInsDbId,
    filter: {
      property: "Date",
      date: { equals: today }
    }
  });
  
  if (existing.results.length > 0) {
    return { exists: true, page: existing.results[0] };
  }
  
  // Get context
  const yesterday = new Date(Date.now() - 86400000).toISOString().split('T')[0];
  
  const [incompleteTasks, todayTasks, activeGoals] = await Promise.all([
    notion.databases.query({
      database_id: tasksDbId,
      filter: {
        and: [
          { property: "Date", date: { equals: yesterday }},
          { property: "Status", select: { does_not_equal: "Done" }}
        ]
      }
    }),
    notion.databases.query({
      database_id: tasksDbId,
      filter: { property: "Date", date: { equals: today }},
      sorts: [{ property: "Time Block", direction: "ascending" }]
    }),
    notion.databases.query({
      database_id: goalsDbId,
      filter: { property: "Status", select: { equals: "In Progress" }},
      sorts: [{ property: "Priority", direction: "ascending" }],
      page_size: 5
    })
  ]);
  
  // Create page
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
          rich_text: [{
            text: {
              content: `Good morning! Today is ${new Date().toLocaleDateString('en-US', {
                weekday: 'long',
                month: 'long',
                day: 'numeric'
              })}`
            }
          }],
          icon: { emoji: "☀️" }
        }
      },
      {
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: "📋 Unfinished from Yesterday" }}]
        }
      },
      ...incompleteTasks.results.map(task => ({
        type: "to_do",
        to_do: {
          rich_text: [{
            text: {
              content: task.properties["Task Name"].title[0].plain_text
            }
          }],
          checked: false
        }
      })),
      {
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: "📅 Today's Schedule" }}]
        }
      },
      ...todayTasks.results.map(task => ({
        type: "bulleted_list_item",
        bulleted_list_item: {
          rich_text: [{
            text: {
              content: `[${task.properties["Time Block"]?.select?.name || "Anytime"}] ${task.properties["Task Name"].title[0].plain_text}`
            }
          }]
        }
      })),
      {
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: "🎯 Active Goals" }}]
        }
      },
      ...activeGoals.results.map(goal => ({
        type: "bulleted_list_item",
        bulleted_list_item: {
          rich_text: [{
            text: {
              content: `${goal.properties.Name.title[0].plain_text} - ${goal.properties.Progress.number}% complete`
            }
          }]
        }
      })),
      {
        type: "divider",
        divider: {}
      },
      {
        type: "heading_2",
        heading_2: {
          rich_text: [{ text: { content: "⭐ Today's Top 3 Priorities" }}],
          color: "orange_background"
        }
      },
      {
        type: "numbered_list_item",
        numbered_list_item: { rich_text: [] }
      },
      {
        type: "numbered_list_item",
        numbered_list_item: { rich_text: [] }
      },
      {
        type: "numbered_list_item",
        numbered_list_item: { rich_text: [] }
      }
    ]
  });
  
  return {
    created: true,
    page: checkIn,
    url: `https://notion.so/${checkIn.id.replace(/-/g, '')}`
  };
}
```

## Workflow 3: Goal Progress Analysis

Comprehensive goal analysis with recommendations.

```javascript
async function analyzeGoalProgress(goalId) {
  // Fetch goal
  const goal = await notion.pages.retrieve({ page_id: goalId });
  const props = goal.properties;
  
  // Calculate metrics
  const startDate = new Date(props["Start Date"].date.start);
  const targetDate = new Date(props["Target Date"].date.start);
  const today = new Date();
  
  const totalDays = Math.floor((targetDate - startDate) / (1000 * 60 * 60 * 24));
  const elapsedDays = Math.floor((today - startDate) / (1000 * 60 * 60 * 24));
  const remainingDays = Math.floor((targetDate - today) / (1000 * 60 * 60 * 24));
  
  const percentTimeElapsed = (elapsedDays / totalDays) * 100;
  const currentProgress = props.Progress.number || 0;
  const expectedProgress = percentTimeElapsed;
  const variance = currentProgress - expectedProgress;
  
  // Determine status
  let status;
  if (variance >= 10) {
    status = {
      level: "ahead",
      emoji: "🚀",
      color: "green",
      message: "You're ahead of schedule!"
    };
  } else if (variance >= -5) {
    status = {
      level: "on-track",
      emoji: "✅",
      color: "blue",
      message: "You're on track!"
    };
  } else if (variance >= -15) {
    status = {
      level: "at-risk",
      emoji: "⚠️",
      color: "yellow",
      message: "Falling behind - action needed"
    };
  } else {
    status = {
      level: "critical",
      emoji: "🔴",
      color: "red",
      message: "Critical - immediate attention required"
    };
  }
  
  // Get action items
  const actionItems = await notion.databases.query({
    database_id: tasksDbId,
    filter: {
      property: "Related Goal",
      relation: { contains: goalId }
    }
  });
  
  const completedCount = actionItems.results.filter(
    t => t.properties.Status.select?.name === "Done"
  ).length;
  
  const totalCount = actionItems.results.length;
  const completionRate = totalCount > 0 ? (completedCount / totalCount) * 100 : 0;
  
  // Generate recommendations
  const recommendations = [];
  
  if (variance < -10) {
    recommendations.push("Break the goal into smaller, actionable tasks");
    recommendations.push("Schedule dedicated time blocks for this goal");
    recommendations.push("Identify and remove blockers");
  }
  
  if (completionRate < 50) {
    recommendations.push("Focus on completing existing action items");
    recommendations.push("Review task priorities");
  }
  
  if (remainingDays < 30) {
    recommendations.push("Increase intensity - less than a month remaining");
    recommendations.push("Review if timeline adjustment is needed");
  }
  
  return {
    goalName: props.Name.title[0].plain_text,
    timeline: {
      startDate: startDate.toLocaleDateString(),
      targetDate: targetDate.toLocaleDateString(),
      daysElapsed: elapsedDays,
      daysRemaining: remainingDays,
      totalDays: totalDays
    },
    progress: {
      current: Math.round(currentProgress),
      expected: Math.round(expectedProgress),
      variance: Math.round(variance)
    },
    actionItems: {
      completed: completedCount,
      total: totalCount,
      completionRate: Math.round(completionRate)
    },
    status: status,
    recommendations: recommendations
  };
}
```

## Workflow 4: Workout Logging

Log workout with personal record checking.

```javascript
async function logWorkout(exercises) {
  // exercises = [
  //   { name: "Bench Press", sets: 3, reps: 8, weight: 185 },
  //   { name: "Squats", sets: 3, reps: 10, weight: 225 }
  // ]
  
  const today = new Date().toISOString().split('T')[0];
  
  // Create workout
  const workout = await notion.pages.create({
    parent: { database_id: workoutsDbId },
    properties: {
      "Workout Name": {
        title: [{ text: { content: "Strength Training" }}]
      },
      "Date": { date: { start: today }},
      "Type": { select: { name: "Strength" }}
    }
  });
  
  const exercisePages = [];
  const personalRecords = [];
  
  // Log each exercise
  for (const ex of exercises) {
    // Create exercise entry
    const exercisePage = await notion.pages.create({
      parent: { database_id: exercisesDbId },
      properties: {
        "Exercise Name": {
          title: [{ text: { content: ex.name }}]
        },
        "Sets": { number: ex.sets },
        "Reps": { number: ex.reps },
        "Weight": { number: ex.weight }
      }
    });
    
    exercisePages.push(exercisePage);
    
    // Check for personal record
    const existingPRs = await notion.databases.query({
      database_id: exercisesDbId,
      filter: {
        and: [
          {
            property: "Exercise Name",
            title: { equals: ex.name }
          },
          {
            property: "Weight",
            number: { greater_than: ex.weight }
          }
        ]
      }
    });
    
    if (existingPRs.results.length === 0) {
      // New PR!
      await notion.pages.update({
        page_id: exercisePage.id,
        properties: {
          "Personal Best Weight": { number: ex.weight },
          "Personal Best Date": { date: { start: today }}
        }
      });
      
      personalRecords.push({
        exercise: ex.name,
        weight: ex.weight
      });
    }
  }
  
  // Link exercises to workout
  await notion.pages.update({
    page_id: workout.id,
    properties: {
      "Exercises": {
        relation: exercisePages.map(ep => ({ id: ep.id }))
      }
    }
  });
  
  return {
    workout: workout,
    exercises: exercisePages,
    personalRecords: personalRecords
  };
}
```

## Workflow 5: Meal Plan Generation

Generate balanced meal plan with shopping list.

```javascript
async function generateMealPlan(days, calorieTarget, proteinTarget) {
  // Query recipes
  const recipes = await notion.databases.query({
    database_id: recipesDbId,
    filter: {
      property: "Calories per Serving",
      number: { less_than_or_equal_to: calorieTarget / 3 }
    }
  });
  
  // Categorize by meal type
  const byType = {
    Breakfast: recipes.results.filter(r => 
      r.properties["Meal Type"].select?.name === "Breakfast"
    ),
    Lunch: recipes.results.filter(r => 
      r.properties["Meal Type"].select?.name === "Lunch"
    ),
    Dinner: recipes.results.filter(r => 
      r.properties["Meal Type"].select?.name === "Dinner"
    ),
    Snack: recipes.results.filter(r => 
      r.properties["Meal Type"].select?.name === "Snack"
    )
  };
  
  // Generate plan
  const plan = [];
  const startDate = new Date();
  startDate.setDate(startDate.getDate() + 1);
  
  for (let day = 0; day < days; day++) {
    const date = new Date(startDate);
    date.setDate(date.getDate() + day);
    const dateStr = date.toISOString().split('T')[0];
    
    const dayMeals = [];
    
    // Select meals
    for (const type of ["Breakfast", "Lunch", "Dinner"]) {
      const options = byType[type];
      if (options.length > 0) {
        const recipe = options[Math.floor(Math.random() * options.length)];
        
        // Create meal entry
        const meal = await notion.pages.create({
          parent: { database_id: mealsDbId },
          properties: {
            "Meal Name": {
              title: [{
                text: {
                  content: recipe.properties["Recipe Name"].title[0].plain_text
                }
              }]
            },
            "Date": { date: { start: dateStr }},
            "Meal Type": { select: { name: type }},
            "Recipes": { relation: [{ id: recipe.id }]}
          }
        });
        
        dayMeals.push(meal);
      }
    }
    
    plan.push({
      date: dateStr,
      meals: dayMeals
    });
  }
  
  // Generate shopping list
  const allIngredients = new Set();
  
  for (const day of plan) {
    for (const meal of day.meals) {
      const mealData = await notion.pages.retrieve({ page_id: meal.id });
      const recipeRelations = mealData.properties.Recipes?.relation || [];
      
      for (const rel of recipeRelations) {
        const recipe = await notion.pages.retrieve({ page_id: rel.id });
        const ingredientRelations = recipe.properties.Ingredients?.relation || [];
        
        for (const ing of ingredientRelations) {
          allIngredients.add(ing.id);
        }
      }
    }
  }
  
  // Create grocery list
  const groceryList = await notion.pages.create({
    parent: { database_id: groceryListDbId },
    properties: {
      "List Name": {
        title: [{
          text: {
            content: `Grocery List - Week of ${plan[0].date}`
          }
        }]
      },
      "Ingredients": {
        relation: Array.from(allIngredients).map(id => ({ id }))
      }
    }
  });
  
  return {
    plan: plan,
    groceryList: groceryList,
    totalMeals: plan.reduce((sum, d) => sum + d.meals.length, 0)
  };
}
```

These are complete, working implementations that use the Notion MCP server to build sophisticated productivity and wellness systems.
