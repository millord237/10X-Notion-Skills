# Complete Database Schemas Reference

This file contains all database schemas with full property definitions, formulas, and rollups.

## Goals Database

### Properties

```json
{
  "Name": {
    "type": "title"
  },
  "Status": {
    "type": "select",
    "options": [
      { "name": "Not Started", "color": "gray" },
      { "name": "In Progress", "color": "blue" },
      { "name": "On Hold", "color": "yellow" },
      { "name": "Completed", "color": "green" },
      { "name": "Abandoned", "color": "red" }
    ]
  },
  "Goal Type": {
    "type": "select",
    "options": [
      { "name": "Annual", "color": "purple" },
      { "name": "Quarterly", "color": "pink" },
      { "name": "Monthly", "color": "orange" },
      { "name": "Weekly", "color": "yellow" },
      { "name": "Daily", "color": "green" }
    ]
  },
  "Progress": {
    "type": "number",
    "format": "percent",
    "description": "0-100%"
  },
  "Start Date": {
    "type": "date"
  },
  "Target Date": {
    "type": "date"
  },
  "Category": {
    "type": "multi_select",
    "options": [
      { "name": "Career", "color": "blue" },
      { "name": "Health", "color": "green" },
      { "name": "Finance", "color": "yellow" },
      { "name": "Personal", "color": "pink" },
      { "name": "Relationships", "color": "red" },
      { "name": "Learning", "color": "purple" }
    ]
  },
  "Priority": {
    "type": "select",
    "options": [
      { "name": "Critical", "color": "red" },
      { "name": "High", "color": "orange" },
      { "name": "Medium", "color": "yellow" },
      { "name": "Low", "color": "gray" }
    ]
  },
  "Parent Goal": {
    "type": "relation",
    "database": "Goals",
    "description": "Link to parent goal for hierarchical structure"
  },
  "Sub-goals": {
    "type": "relation",
    "database": "Goals",
    "description": "Linked child goals"
  },
  "Action Items": {
    "type": "relation",
    "database": "Tasks",
    "description": "Tasks related to this goal"
  },
  "Related Workouts": {
    "type": "relation",
    "database": "Workouts",
    "description": "Workouts contributing to this goal"
  },
  "Related Check-ins": {
    "type": "relation",
    "database": "Daily Check-ins",
    "description": "Daily check-ins mentioning this goal"
  },
  "Days Remaining": {
    "type": "formula",
    "formula": "dateBetween(prop(\"Target Date\"), now(), \"days\")"
  },
  "Days Total": {
    "type": "formula",
    "formula": "dateBetween(prop(\"Target Date\"), prop(\"Start Date\"), \"days\")"
  },
  "Days Elapsed": {
    "type": "formula",
    "formula": "dateBetween(now(), prop(\"Start Date\"), \"days\")"
  },
  "Time Elapsed %": {
    "type": "formula",
    "formula": "if(empty(prop(\"Start Date\")) or empty(prop(\"Target Date\")), 0, (dateBetween(now(), prop(\"Start Date\"), \"days\") / dateBetween(prop(\"Target Date\"), prop(\"Start Date\"), \"days\")) * 100)"
  },
  "Expected Progress": {
    "type": "formula",
    "formula": "prop(\"Time Elapsed %\")"
  },
  "Progress Variance": {
    "type": "formula",
    "formula": "prop(\"Progress\") - prop(\"Expected Progress\")"
  },
  "On Track": {
    "type": "formula",
    "formula": "if(prop(\"Progress Variance\") >= 10, \"🚀 Ahead\", if(prop(\"Progress Variance\") >= -5, \"✅ On Track\", if(prop(\"Progress Variance\") >= -15, \"⚠️ At Risk\", \"🔴 Critical\")))"
  },
  "Completion Rate": {
    "type": "formula",
    "formula": "if(prop(\"Days Elapsed\") > 0, prop(\"Progress\") / prop(\"Days Elapsed\"), 0)"
  },
  "Required Daily Rate": {
    "type": "formula",
    "formula": "if(prop(\"Days Remaining\") > 0, (100 - prop(\"Progress\")) / prop(\"Days Remaining\"), 0)"
  },
  "Notes": {
    "type": "rich_text"
  },
  "Last Reviewed": {
    "type": "date"
  }
}
```

### Views

1. **Dashboard** (Gallery)
   - Group by: Status
   - Filter: Status != Completed, Status != Abandoned
   - Sort: Priority (ascending), Target Date (ascending)

2. **Timeline** (Timeline)
   - Start date: Start Date
   - End date: Target Date
   - Color by: On Track formula

3. **By Category** (Board)
   - Group by: Category
   - Filter: Status = In Progress

4. **At Risk** (Table)
   - Filter: Progress Variance < -10, Status = In Progress
   - Sort: Progress Variance (ascending)

5. **Review Queue** (Table)
   - Filter: Last Reviewed is empty OR Last Reviewed before 7 days ago
   - Sort: Priority (ascending)

---

## Tasks Database

### Properties

```json
{
  "Task Name": {
    "type": "title"
  },
  "Status": {
    "type": "select",
    "options": [
      { "name": "To Do", "color": "gray" },
      { "name": "In Progress", "color": "blue" },
      { "name": "Done", "color": "green" },
      { "name": "Blocked", "color": "red" },
      { "name": "Cancelled", "color": "red" }
    ]
  },
  "Priority": {
    "type": "select",
    "options": [
      { "name": "Critical", "color": "red" },
      { "name": "High", "color": "orange" },
      { "name": "Medium", "color": "yellow" },
      { "name": "Low", "color": "gray" }
    ]
  },
  "Date": {
    "type": "date",
    "description": "Due date"
  },
  "Time Block": {
    "type": "select",
    "options": [
      { "name": "Morning", "color": "blue" },
      { "name": "Midday", "color": "green" },
      { "name": "Afternoon", "color": "orange" },
      { "name": "Evening", "color": "purple" },
      { "name": "Anytime", "color": "gray" }
    ]
  },
  "Estimated Duration": {
    "type": "number",
    "description": "In minutes"
  },
  "Actual Duration": {
    "type": "number",
    "description": "In minutes"
  },
  "Duration Variance": {
    "type": "formula",
    "formula": "if(empty(prop(\"Actual Duration\")), 0, prop(\"Actual Duration\") - prop(\"Estimated Duration\"))"
  },
  "Energy Level Required": {
    "type": "select",
    "options": [
      { "name": "High", "color": "red" },
      { "name": "Medium", "color": "yellow" },
      { "name": "Low", "color": "green" }
    ]
  },
  "Related Goal": {
    "type": "relation",
    "database": "Goals"
  },
  "Related Project": {
    "type": "relation",
    "database": "Projects"
  },
  "Tags": {
    "type": "multi_select",
    "options": [
      { "name": "Quick", "color": "green" },
      { "name": "Focus", "color": "blue" },
      { "name": "Email", "color": "gray" },
      { "name": "Meeting", "color": "purple" },
      { "name": "Admin", "color": "yellow" }
    ]
  },
  "Overdue": {
    "type": "formula",
    "formula": "if(empty(prop(\"Date\")), false, if(prop(\"Status\") != \"Done\" and prop(\"Status\") != \"Cancelled\", dateBetween(now(), prop(\"Date\"), \"days\") > 0, false))"
  },
  "Days Until Due": {
    "type": "formula",
    "formula": "if(empty(prop(\"Date\")), 0, dateBetween(prop(\"Date\"), now(), \"days\"))"
  }
}
```

---

## Workouts Database

### Properties

```json
{
  "Workout Name": {
    "type": "title"
  },
  "Date": {
    "type": "date"
  },
  "Type": {
    "type": "select",
    "options": [
      { "name": "Cardio", "color": "red" },
      { "name": "Strength", "color": "blue" },
      { "name": "Flexibility", "color": "green" },
      { "name": "Sports", "color": "orange" },
      { "name": "Recovery", "color": "purple" },
      { "name": "Other", "color": "gray" }
    ]
  },
  "Duration": {
    "type": "number",
    "description": "In minutes"
  },
  "Intensity": {
    "type": "select",
    "options": [
      { "name": "1 - Very Light", "color": "gray" },
      { "name": "2 - Light", "color": "gray" },
      { "name": "3 - Light-Moderate", "color": "yellow" },
      { "name": "4 - Moderate", "color": "yellow" },
      { "name": "5 - Moderate", "color": "yellow" },
      { "name": "6 - Moderate-High", "color": "orange" },
      { "name": "7 - High", "color": "orange" },
      { "name": "8 - High", "color": "red" },
      { "name": "9 - Very High", "color": "red" },
      { "name": "10 - Maximum", "color": "red" }
    ]
  },
  "Muscle Groups": {
    "type": "multi_select",
    "options": [
      { "name": "Chest", "color": "red" },
      { "name": "Back", "color": "blue" },
      { "name": "Legs", "color": "green" },
      { "name": "Shoulders", "color": "orange" },
      { "name": "Arms", "color": "purple" },
      { "name": "Core", "color": "yellow" },
      { "name": "Full Body", "color": "pink" }
    ]
  },
  "Calories Burned": {
    "type": "number"
  },
  "Exercises": {
    "type": "relation",
    "database": "Exercises",
    "description": "Exercises performed in this workout"
  },
  "Total Volume": {
    "type": "rollup",
    "relation": "Exercises",
    "property": "Volume",
    "function": "sum",
    "description": "Total volume (sets × reps × weight) for all exercises"
  },
  "Total Sets": {
    "type": "rollup",
    "relation": "Exercises",
    "property": "Sets",
    "function": "sum"
  },
  "Mood Rating": {
    "type": "select",
    "options": [
      { "name": "Excellent", "color": "green" },
      { "name": "Good", "color": "blue" },
      { "name": "Neutral", "color": "gray" },
      { "name": "Tired", "color": "yellow" },
      { "name": "Exhausted", "color": "red" }
    ]
  },
  "Related Goal": {
    "type": "relation",
    "database": "Goals"
  },
  "Related Program": {
    "type": "relation",
    "database": "Workout Programs"
  },
  "Notes": {
    "type": "rich_text"
  },
  "Rest Days Since Last": {
    "type": "formula",
    "formula": "dateBetween(now(), prop(\"Date\"), \"days\")"
  }
}
```

---

## Exercises Database

### Properties

```json
{
  "Exercise Name": {
    "type": "title"
  },
  "Type": {
    "type": "select",
    "options": [
      { "name": "Cardio", "color": "red" },
      { "name": "Strength", "color": "blue" },
      { "name": "Flexibility", "color": "green" },
      { "name": "Plyometric", "color": "orange" }
    ]
  },
  "Primary Muscle": {
    "type": "select",
    "options": [
      { "name": "Chest", "color": "red" },
      { "name": "Back", "color": "blue" },
      { "name": "Legs", "color": "green" },
      { "name": "Shoulders", "color": "orange" },
      { "name": "Arms", "color": "purple" },
      { "name": "Core", "color": "yellow" },
      { "name": "Cardio", "color": "red" }
    ]
  },
  "Secondary Muscles": {
    "type": "multi_select",
    "options": [
      { "name": "Chest", "color": "red" },
      { "name": "Back", "color": "blue" },
      { "name": "Legs", "color": "green" },
      { "name": "Shoulders", "color": "orange" },
      { "name": "Arms", "color": "purple" },
      { "name": "Core", "color": "yellow" }
    ]
  },
  "Equipment": {
    "type": "multi_select",
    "options": [
      { "name": "Barbell", "color": "gray" },
      { "name": "Dumbbell", "color": "gray" },
      { "name": "Machine", "color": "blue" },
      { "name": "Cable", "color": "purple" },
      { "name": "Bodyweight", "color": "green" },
      { "name": "Bands", "color": "orange" },
      { "name": "Kettlebell", "color": "red" },
      { "name": "Medicine Ball", "color": "yellow" }
    ]
  },
  "Sets": {
    "type": "number"
  },
  "Reps": {
    "type": "number"
  },
  "Weight": {
    "type": "number",
    "description": "In pounds"
  },
  "Volume": {
    "type": "formula",
    "formula": "prop(\"Sets\") * prop(\"Reps\") * prop(\"Weight\")",
    "description": "Total volume = sets × reps × weight"
  },
  "Personal Best Weight": {
    "type": "number"
  },
  "Personal Best Date": {
    "type": "date"
  },
  "Personal Best Reps": {
    "type": "number"
  },
  "Rest Time": {
    "type": "number",
    "description": "Rest between sets in seconds"
  },
  "Tempo": {
    "type": "text",
    "description": "e.g., 3-0-1-0 (eccentric-pause-concentric-pause)"
  },
  "Form Notes": {
    "type": "rich_text"
  },
  "Video URL": {
    "type": "url"
  },
  "Last Performed": {
    "type": "date"
  },
  "Times Performed": {
    "type": "number"
  }
}
```

---

## Meals Database

### Properties

```json
{
  "Meal Name": {
    "type": "title"
  },
  "Date": {
    "type": "date"
  },
  "Meal Type": {
    "type": "select",
    "options": [
      { "name": "Breakfast", "color": "yellow" },
      { "name": "Lunch", "color": "orange" },
      { "name": "Dinner", "color": "red" },
      { "name": "Snack", "color": "green" }
    ]
  },
  "Recipes": {
    "type": "relation",
    "database": "Recipes"
  },
  "Total Calories": {
    "type": "rollup",
    "relation": "Recipes",
    "property": "Calories per Serving",
    "function": "sum"
  },
  "Total Protein": {
    "type": "rollup",
    "relation": "Recipes",
    "property": "Protein",
    "function": "sum"
  },
  "Total Carbs": {
    "type": "rollup",
    "relation": "Recipes",
    "property": "Carbs",
    "function": "sum"
  },
  "Total Fats": {
    "type": "rollup",
    "relation": "Recipes",
    "property": "Fats",
    "function": "sum"
  },
  "Protein %": {
    "type": "formula",
    "formula": "if(prop(\"Total Calories\") > 0, (prop(\"Total Protein\") * 4 / prop(\"Total Calories\")) * 100, 0)"
  },
  "Carbs %": {
    "type": "formula",
    "formula": "if(prop(\"Total Calories\") > 0, (prop(\"Total Carbs\") * 4 / prop(\"Total Calories\")) * 100, 0)"
  },
  "Fats %": {
    "type": "formula",
    "formula": "if(prop(\"Total Calories\") > 0, (prop(\"Total Fats\") * 9 / prop(\"Total Calories\")) * 100, 0)"
  },
  "Macro Balance": {
    "type": "formula",
    "formula": "format(round(prop(\"Protein %\"))) + \"P / \" + format(round(prop(\"Carbs %\"))) + \"C / \" + format(round(prop(\"Fats %\"))) + \"F\""
  },
  "Rating": {
    "type": "select",
    "options": [
      { "name": "😍 Loved it", "color": "green" },
      { "name": "😊 Really good", "color": "blue" },
      { "name": "😐 It was okay", "color": "gray" },
      { "name": "😕 Not great", "color": "yellow" },
      { "name": "😫 Didn't like", "color": "red" }
    ]
  },
  "Notes": {
    "type": "rich_text"
  }
}
```

---

## Recipes Database

### Properties

```json
{
  "Recipe Name": {
    "type": "title"
  },
  "Cuisine Type": {
    "type": "select",
    "options": [
      { "name": "American", "color": "red" },
      { "name": "Italian", "color": "green" },
      { "name": "Mexican", "color": "orange" },
      { "name": "Asian", "color": "yellow" },
      { "name": "Mediterranean", "color": "blue" },
      { "name": "Indian", "color": "purple" },
      { "name": "Other", "color": "gray" }
    ]
  },
  "Meal Type": {
    "type": "select",
    "options": [
      { "name": "Breakfast", "color": "yellow" },
      { "name": "Lunch", "color": "orange" },
      { "name": "Dinner", "color": "red" },
      { "name": "Snack", "color": "green" },
      { "name": "Dessert", "color": "pink" }
    ]
  },
  "Servings": {
    "type": "number"
  },
  "Prep Time": {
    "type": "number",
    "description": "In minutes"
  },
  "Cook Time": {
    "type": "number",
    "description": "In minutes"
  },
  "Total Time": {
    "type": "formula",
    "formula": "prop(\"Prep Time\") + prop(\"Cook Time\")"
  },
  "Calories per Serving": {
    "type": "number"
  },
  "Protein": {
    "type": "number",
    "description": "Grams per serving"
  },
  "Carbs": {
    "type": "number",
    "description": "Grams per serving"
  },
  "Fats": {
    "type": "number",
    "description": "Grams per serving"
  },
  "Fiber": {
    "type": "number",
    "description": "Grams per serving"
  },
  "Sugar": {
    "type": "number",
    "description": "Grams per serving"
  },
  "Ingredients": {
    "type": "relation",
    "database": "Ingredients"
  },
  "Instructions": {
    "type": "rich_text"
  },
  "Tags": {
    "type": "multi_select",
    "options": [
      { "name": "High Protein", "color": "blue" },
      { "name": "Low Carb", "color": "green" },
      { "name": "Vegetarian", "color": "green" },
      { "name": "Vegan", "color": "green" },
      { "name": "Gluten Free", "color": "yellow" },
      { "name": "Dairy Free", "color": "yellow" },
      { "name": "Quick", "color": "orange" },
      { "name": "Budget", "color": "gray" },
      { "name": "Meal Prep", "color": "purple" }
    ]
  },
  "Cost per Serving": {
    "type": "number",
    "format": "dollar"
  },
  "Photos": {
    "type": "files"
  },
  "Source": {
    "type": "url"
  },
  "Times Made": {
    "type": "number"
  },
  "Last Made": {
    "type": "date"
  },
  "Average Rating": {
    "type": "number",
    "description": "1-5 stars"
  }
}
```

---

## Daily Check-ins Database

### Properties

```json
{
  "Date": {
    "type": "date",
    "description": "Unique - one check-in per day"
  },
  "Morning Energy": {
    "type": "select",
    "options": [
      { "name": "High", "color": "green" },
      { "name": "Medium", "color": "yellow" },
      { "name": "Low", "color": "red" }
    ]
  },
  "Top 3 Tasks": {
    "type": "rich_text"
  },
  "Evening Mood": {
    "type": "select",
    "options": [
      { "name": "Excellent", "color": "green" },
      { "name": "Good", "color": "blue" },
      { "name": "Neutral", "color": "gray" },
      { "name": "Low", "color": "yellow" },
      { "name": "Poor", "color": "red" }
    ]
  },
  "What Went Well": {
    "type": "rich_text"
  },
  "Challenges Faced": {
    "type": "rich_text"
  },
  "Gratitude": {
    "type": "rich_text",
    "description": "Three things I'm grateful for"
  },
  "Tomorrow's Focus": {
    "type": "rich_text"
  },
  "Workout Completed": {
    "type": "checkbox"
  },
  "Nutrition Goal Met": {
    "type": "checkbox"
  },
  "Sleep Hours": {
    "type": "number"
  },
  "Sleep Quality": {
    "type": "select",
    "options": [
      { "name": "Excellent", "color": "green" },
      { "name": "Good", "color": "blue" },
      { "name": "Fair", "color": "yellow" },
      { "name": "Poor", "color": "red" }
    ]
  },
  "Water Intake": {
    "type": "number",
    "description": "Glasses of water"
  },
  "Steps": {
    "type": "number"
  },
  "Meditation Minutes": {
    "type": "number"
  },
  "Screen Time Hours": {
    "type": "number"
  },
  "Overall Rating": {
    "type": "select",
    "options": [
      { "name": "10", "color": "green" },
      { "name": "9", "color": "green" },
      { "name": "8", "color": "blue" },
      { "name": "7", "color": "blue" },
      { "name": "6", "color": "gray" },
      { "name": "5", "color": "gray" },
      { "name": "4", "color": "yellow" },
      { "name": "3", "color": "yellow" },
      { "name": "2", "color": "red" },
      { "name": "1", "color": "red" }
    ]
  },
  "Habits Completed": {
    "type": "relation",
    "database": "Habits"
  },
  "Habits Count": {
    "type": "rollup",
    "relation": "Habits Completed",
    "property": "Habit Name",
    "function": "count"
  },
  "Tasks Completed": {
    "type": "relation",
    "database": "Tasks",
    "filter": "Status = Done"
  },
  "Tasks Count": {
    "type": "rollup",
    "relation": "Tasks Completed",
    "property": "Task Name",
    "function": "count"
  },
  "Related Goals": {
    "type": "relation",
    "database": "Goals"
  },
  "Productivity Score": {
    "type": "formula",
    "formula": "if(empty(prop(\"Overall Rating\")), 0, toNumber(prop(\"Overall Rating\")) * (prop(\"Tasks Count\") / 10))"
  },
  "Wellness Score": {
    "type": "formula",
    "formula": "(prop(\"Sleep Hours\") / 8) * 100 * if(prop(\"Workout Completed\"), 1.2, 1) * if(prop(\"Nutrition Goal Met\"), 1.2, 1)"
  }
}
```

---

## Habits Database

### Properties

```json
{
  "Habit Name": {
    "type": "title"
  },
  "Category": {
    "type": "select",
    "options": [
      { "name": "Health", "color": "green" },
      { "name": "Productivity", "color": "blue" },
      { "name": "Learning", "color": "purple" },
      { "name": "Social", "color": "pink" },
      { "name": "Self-Care", "color": "yellow" },
      { "name": "Finance", "color": "orange" }
    ]
  },
  "Frequency Goal": {
    "type": "select",
    "options": [
      { "name": "Daily", "color": "red" },
      { "name": "5x per week", "color": "orange" },
      { "name": "3x per week", "color": "yellow" },
      { "name": "Weekly", "color": "blue" },
      { "name": "Monthly", "color": "gray" }
    ]
  },
  "Current Streak": {
    "type": "number",
    "description": "Consecutive days/weeks"
  },
  "Longest Streak": {
    "type": "number"
  },
  "Total Completions": {
    "type": "number"
  },
  "Last Completed": {
    "type": "date"
  },
  "Related Goal": {
    "type": "relation",
    "database": "Goals"
  },
  "Check-ins": {
    "type": "relation",
    "database": "Daily Check-ins"
  },
  "Completion Rate": {
    "type": "number",
    "format": "percent",
    "description": "% of days completed"
  },
  "Notes": {
    "type": "rich_text"
  },
  "Started": {
    "type": "date"
  },
  "Days Since Start": {
    "type": "formula",
    "formula": "dateBetween(now(), prop(\"Started\"), \"days\")"
  }
}
```

---

## Common Formula Patterns

### Progress Tracking
```
Expected Progress = (Days Elapsed / Total Days) * 100
Progress Variance = Actual Progress - Expected Progress
On Track = if(Variance >= -5, "✅", "⚠️")
```

### Time Calculations
```
Days Remaining = dateBetween(Target Date, now(), "days")
Days Elapsed = dateBetween(now(), Start Date, "days")
Weeks Remaining = dateBetween(Target Date, now(), "weeks")
```

### Macro Calculations
```
Protein % = (Protein grams * 4 / Total Calories) * 100
Carbs % = (Carbs grams * 4 / Total Calories) * 100
Fats % = (Fats grams * 9 / Total Calories) * 100
```

### Volume Tracking
```
Volume = Sets × Reps × Weight
Total Volume = sum(all exercises)
Average Volume = mean(volume per session)
```

### Streak Tracking
```
Current Streak = count(consecutive days)
Longest Streak = max(all streaks)
Completion Rate = (Total Completions / Days Since Start) * 100
```

This reference provides complete schemas for all databases with proper property types, formulas, and relations.
