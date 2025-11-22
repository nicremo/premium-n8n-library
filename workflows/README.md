# Workflows

This is where all the actual workflows live, organized by category.

## Adding a Workflow

When adding a workflow to this repo:

1. **Choose the right category** - Put it where it makes sense. If it doesn't fit anywhere, maybe we need a new category.

2. **Name it descriptively** - Use this format: `service-name_what-it-does.json`
   - Good: `slack_daily-standup-reminder.json`
   - Bad: `workflow1.json` or `my_thing.json`

3. **Add a comment at the top** - n8n workflows support notes. Add a brief description of what it does, what credentials it needs, and any setup steps.

4. **Test it first** - Obviously. Don't commit broken stuff.

## Categories

- **automation/** - General workflow automation
- **ai/** - Anything involving AI/ML APIs
- **data-processing/** - ETL, data transformation, etc.
- **integrations/** - Connecting different services
- **communication/** - Messaging, email, notifications
- **productivity/** - Task management, scheduling, etc.

If you think a category should exist that doesn't, just create it. This isn't bureaucratic.
