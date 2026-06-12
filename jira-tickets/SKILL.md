---
name: jira-tickets
description: >-
  Create JIRA tickets at any hierarchy level (Epic, Story, Task, Sub-task, Bug).
  Infers as much as possible from conversation context and recommends values;
  asks the user only for criteria it cannot determine. Use when the user asks to
  create a JIRA ticket, issue, story, task, bug, epic, or sub-task, or mentions
  filing/logging work in JIRA.
---

# JIRA Ticket Creation

Create JIRA tickets via the GitKraken MCP `issues_create` tool with `provider: "jira"`.

## Core Principle

**Infer first, confirm second, ask last.**

1. Extract every possible field from conversation context (task description, type, labels, etc.)
2. Present inferred values as **(Recommended)** options using `AskQuestion`
3. Only ask the user about fields you genuinely cannot determine

## Workflow

### Step 1: Gather Context

Before asking the user anything, scan the current conversation for:
- What work is being discussed (determines **title** and **description**)
- Whether it's a feature, bug fix, refactor, etc. (determines **issue type**)
- Scale of work (determines **hierarchy level**: Epic > Story > Task > Sub-task)
- Any mentioned project names or keys (determines **project key**)
- Any mentioned people (determines **assignee**)

### Step 2: Confirm with the User

Use `AskQuestion` to present a single confirmation form. Group everything into one call where possible to avoid back-and-forth.

**Always ask for:**
- JIRA Project Key (the user works across multiple projects)
- Assignee

**Ask with a recommended default when you can infer:**
- Issue type (Epic / Story / Task / Sub-task / Bug)
- Title
- Labels

**Auto-fill without asking when obvious:**
- Description body (compose from conversation context)

#### Issue Type Guidance

Use this to pick a recommended type from context:

| Signal in conversation | Recommended type |
|------------------------|-----------------|
| Large initiative, multiple features, quarter-level goal | Epic |
| User-facing feature, "as a user I want..." | Story |
| General work item, technical task, chore | Task |
| Breakdown of an already-discussed Story or Task | Sub-task |
| Something is broken, regression, unexpected behaviour | Bug |

### Step 3: Compose the Ticket

Build the `issues_create` call:

```
CallMcpTool:
  server: user-eamodio.gitlens-extension-GitKraken
  toolName: issues_create
  arguments:
    provider: "jira"
    repository_name: "<PROJECT_KEY>"    # e.g. "ENG"
    title: "<TITLE>"
    body: "<DESCRIPTION in markdown>"
    assignees: ["<assignee>"]           # omit if unassigned
    labels: ["<label1>", "<label2>"]    # omit if none
```

### Step 4: Confirm Success

After the MCP call returns, report the created ticket key/URL back to the user.

## Description Template

When composing the ticket body, use this structure (adapt as needed):

```markdown
## Summary
[1-2 sentence overview]

## Context
[Why this work is needed - pull from conversation]

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]

## Notes
[Any additional context, links, or technical details]
```

For **Bugs**, use this variant instead:

```markdown
## Summary
[What is broken]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]

## Expected Behaviour
[What should happen]

## Actual Behaviour
[What actually happens]

## Notes
[Environment, logs, screenshots, etc.]
```

## Batch Creation

When the user describes multiple tasks at once (e.g. "create tickets for X, Y, and Z"):

1. Present all tickets as a table for review before creating any
2. Include columns: Type, Title, Project Key, Assignee
3. Ask for confirmation, then create them sequentially
4. Report all created ticket keys at the end

## AskQuestion Format Guide

Structure your `AskQuestion` calls to minimise friction. Example:

```
AskQuestion:
  title: "Create JIRA Ticket"
  questions:
    - id: project_key
      prompt: "Which JIRA project?"
      options: [likely projects or "Other"]
    - id: issue_type
      prompt: "Issue type?"
      options:
        - "Task (Recommended)"   # put inferred type first
        - "Story"
        - "Bug"
        - "Epic"
        - "Sub-task"
    - id: assignee
      prompt: "Assign to?"
      options:
        - "Me (Recommended)"
        - "Unassigned"
    - id: confirm_title
      prompt: "Title: '<inferred title>' - looks good?"
      options:
        - "Yes (Recommended)"
        - "No, I'll rephrase"
```

## Important Notes

- The `repository_name` field is the **JIRA project key** (e.g. `ENG`, `PROJ`), not a repo name.
- The MCP tool does not support setting issue type directly via a field; include the type in the title prefix or description if your JIRA workflow requires it (e.g. `[Bug] Title here`).
- If the MCP call fails with an auth error, tell the user to check their GitKraken JIRA integration in the GitLens extension settings.
- Always present the created ticket details back to the user after successful creation.
