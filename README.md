# Cursor Agent Skills

A collection of reusable [Cursor](https://cursor.com) agent skills.

## Skills

### JIRA Ticket Creation

**Path:** `jira-tickets/SKILL.md`

Create JIRA tickets at any hierarchy level (Epic, Story, Task, Sub-task, Bug) via the GitKraken MCP integration. The skill infers as much as possible from conversation context and only asks for what it can't determine.

**Requirements:**
- [GitLens extension](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) with a connected JIRA integration
- GitKraken MCP server enabled in Cursor

**Usage:**
1. Copy `jira-tickets/SKILL.md` into your project's `.cursor/skills/jira-tickets/` directory
2. Ask the agent to create a JIRA ticket — it will pick up the skill automatically

## License

MIT
