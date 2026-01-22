# Notion Comprehensive System - Claude Code Skill

**Transform your Notion into an intelligent productivity and wellness command center!**

This is a complete, production-ready Claude Code Skill that enables Claude to build and manage sophisticated, interconnected Notion systems.

## 🎯 What This Skill Does

Creates complete Notion systems with:

✅ **Goal Tracking** - Multi-level goals with automatic progress analytics  
✅ **Fitness Logging** - Workout programs, exercise library, body measurements  
✅ **Nutrition Planning** - Meal plans with macro tracking and grocery lists  
✅ **Daily Check-ins** - Morning planning and evening reflection  
✅ **Habit Tracking** - Streak tracking and consistency analytics  
✅ **Cross-Database Analytics** - Insights across all your systems  

**Everything is interconnected** - Goals link to tasks, workouts link to goals, meals track towards nutrition targets, check-ins reference everything.

## 📦 Installation

### Step 1: Install Notion MCP Server

Edit your Claude Code config file:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`
- **Cursor**: `~/.cursor/mcp.json`

Add:
```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "NOTION_TOKEN": "YOUR_TOKEN_HERE"
      }
    }
  }
}
```

### Step 2: Get Notion Token

1. Visit https://www.notion.so/profile/integrations
2. Click **"+ New integration"**
3. Name: "Claude System"
4. Capabilities: ✅ Read, ✅ Update, ✅ Insert
5. Copy token (starts with `ntn_`)
6. Paste into config above

### Step 3: Connect Workspace

1. In Notion, go to your workspace page
2. Click **"..."** → **"Connections"**
3. Add your integration
4. **Restart Claude Code**

### Step 4: Install This Skill

**Personal Installation:**
```bash
# Extract the zip
unzip notion-comprehensive-system-skill.zip

# Copy to personal skills
cp -r notion-comprehensive-system ~/.claude/skills/
```

**Project Installation (Team):**
```bash
# Copy to project skills
cp -r notion-comprehensive-system .claude/skills/

# Commit to git
git add .claude/skills/
git commit -m "Add Notion comprehensive system skill"
git push
```

### Step 5: Verify Installation

Restart Claude Code, then ask:
```
What skills are available?
```

You should see **"notion-comprehensive-system"** in the list.

## 🚀 Quick Start

### First Use

```
"Set up a complete productivity and wellness system in Notion"
```

I will:
1. Ask what systems you need (goals, fitness, nutrition, etc.)
2. Create 12+ interconnected databases
3. Set up all relations and formulas
4. Configure multiple views per database
5. Generate template pages
6. Create a dashboard
7. Provide usage guide

### Daily Operations

**Morning Planning:**
```
"Create my morning check-in"
```

**Goal Management:**
```
"Create a goal to lose 20 pounds in 4 months"
"How am I doing on my 2025 goals?"
"Show me goals that need attention"
```

**Fitness Tracking:**
```
"Log workout: bench press 3x8 at 185lbs, squats 3x10 at 225lbs"
"Show my workout progress this month"
"Create a 12-week marathon training program"
```

**Nutrition:**
```
"Generate a meal plan for next week with 2000 calories and 150g protein"
"Add a recipe: Chicken Stir-Fry - 400 cal, 35g protein"
"Create grocery list from this week's meals"
```

**Analytics:**
```
"Show correlation between my sleep and productivity"
"Generate my monthly progress report"
"What habits am I most consistent with?"
```

## 🏗️ What Gets Created

### Productivity System
- **Goals** - Hierarchical goals with progress formulas
- **Projects** - Multi-phase projects
- **Tasks** - Action items with time-blocking
- **Daily Plans** - Time-blocking system

### Fitness System
- **Workouts** - Exercise logging with volume tracking
- **Exercises** - Exercise library with personal records
- **Body Measurements** - Progress tracking
- **Programs** - Structured training plans

### Nutrition System
- **Meals** - Meal logging with macro tracking
- **Recipes** - Recipe library with nutritional info
- **Ingredients** - Ingredient database
- **Grocery Lists** - Auto-generated from meal plans

### Wellness System
- **Daily Check-ins** - Morning and evening reflection
- **Habits** - Streak tracking
- **Mood Logs** - Pattern analysis
- **Sleep Tracker** - Quality metrics

## 🔗 Smart Features

### Automatic Sub-Goal Generation

When you create a goal like "Lose 20 pounds in 4 months", I automatically:
- Generate 16 weekly sub-goals
- Calculate required weekly progress (1.25 lbs/week)
- Create action items for each phase
- Link to workout programs
- Set up progress tracking

### Progressive Overload Tracking

For fitness goals, I track:
- Volume progression (sets × reps × weight)
- Personal record improvements
- Rest day patterns
- Muscle group frequency
- Intensity trends

### Macro-Balanced Meal Planning

When generating meal plans, I:
- Balance daily macros (protein/carbs/fats)
- Vary cuisines and meal types
- Calculate per-serving nutrition
- Generate shopping lists automatically
- Track meal ratings for future planning

### Cross-Database Analytics

I analyze patterns like:
- Sleep hours vs. task completion rates
- Workout frequency vs. goal progress
- Nutrition adherence vs. energy levels
- Habit consistency vs. overall ratings

## 📊 Database Schemas

All databases include:

**Smart Relations:**
- Goals → Projects → Tasks
- Workouts → Exercises
- Meals → Recipes → Ingredients
- Check-ins → Habits → Goals

**Automated Formulas:**
- Progress tracking (expected vs. actual)
- Days remaining calculations
- Macro percentages (P/C/F)
- Volume calculations (sets × reps × weight)
- Streak counting

**Multiple Views:**
- Dashboard (gallery/board)
- Calendar (timeline)
- Detail (table)
- Analysis (filtered insights)

See [reference/database-schemas.md](reference/database-schemas.md) for complete schemas.

## 💡 Example Workflows

### Creating a Marathon Training Goal

```
You: "Create a goal to run a marathon in 6 months"

I will:
1. Create main goal: "Run Marathon"
2. Generate 24 weekly sub-goals
3. Create training program database entry
4. Generate progressive running schedule
5. Link workout templates
6. Set up progress tracking
7. Create measurement milestones
```

### Daily Check-In Flow

```
You: "Create my morning check-in"

I will:
1. Check if today's check-in exists
2. Query yesterday's uncompleted tasks
3. Get today's schedule
4. Fetch active goals for reference
5. Create structured page with:
   - Morning planning section
   - Today's priorities
   - Evening reflection prompts
   - Habit tracking checkboxes
```

### Analyzing Goal Progress

```
You: "How am I doing on my Q1 goals?"

I will:
1. Query all Q1 goals
2. Calculate for each:
   - Time elapsed vs. total time
   - Expected progress vs. actual
   - Progress variance
   - Days remaining
3. Categorize by status:
   - 🚀 Ahead (variance > +10%)
   - ✅ On Track (variance > -5%)
   - ⚠️ At Risk (variance > -15%)
   - 🔴 Critical (variance < -15%)
4. Generate report with recommendations
5. Create action items for at-risk goals
```

## 🔧 Customization

After initial setup, you can:

1. **Add Properties** - Extend any database
2. **Create Views** - Custom filters and groupings
3. **Modify Formulas** - Adjust calculations
4. **Create Templates** - Your own page templates
5. **Add Databases** - New tracking systems
6. **Build Dashboards** - Custom analytics views

Just ask Claude:
```
"Add a 'Difficulty' property to my exercises database"
"Create a view showing overdue high-priority tasks"
"Add a weekly review template"
```

## 📚 Documentation

- **[SKILL.md](SKILL.md)** - Complete skill documentation with implementations
- **[reference/database-schemas.md](reference/database-schemas.md)** - All database schemas
- **[examples/](examples/)** - Working code examples
- **[templates/](templates/)** - Database and view templates

## 🐛 Troubleshooting

### "Cannot access Notion"
→ Verify MCP server in config  
→ Check token is valid  
→ Restart Claude Code

### "Database not found"
→ Connect integration to Notion page  
→ Check integration permissions  
→ Verify databases created

### "Formula error"
→ Formulas need null checks (I handle this)  
→ Check property names match

### Claude doesn't use skill
→ Be specific: "Create goal tracking in Notion"  
→ Not vague: "Help with Notion"

## 🎓 Best Practices

1. **Start Simple** - Begin with goals only, add more later
2. **Use Relations** - Connect everything for insights
3. **Regular Reviews** - Weekly check-ins and monthly reports
4. **Customize Freely** - Make it yours
5. **Ask Claude** - I know the system structure

## 📝 Version History

- **v1.0.0** (2025-01-12)
  - Initial release
  - Complete database schemas
  - All core workflows
  - Cross-database analytics
  - Template system

## 📄 License

MIT License - Use and modify freely

## 🙏 Credits

**Created for OpenAnalyst**  
Built with Claude Code and Notion API  
Following official Claude Code Skill specification

---

## 🚀 Ready to Transform Your Notion?

Just ask Claude:

```
"Set up my complete Notion system"
```

And watch your productivity workspace come to life! 🎯
