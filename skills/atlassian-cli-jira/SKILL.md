---
name: atlassian-cli-jira
description: Common workflows for Atlassian CLI (acli) Jira commands
---

# Atlassian CLI - Jira Workflows

Quick reference for common Atlassian CLI Jira operations. Requires `acli` to be installed and authenticated.

**Documentation:** https://developer.atlassian.com/cloud/acli/reference/commands/jira/

## Prerequisites

If `acli` is not installed or not authenticated, instruct the user to run:

```bash
# Authenticate (opens browser for OAuth)
acli auth login --web

# Verify authentication
acli auth status
```

## Common JQL Patterns

Use these JQL queries with `acli jira workitem search --jql "..."`:

### Find tickets assigned to you

```bash
acli jira workitem search --jql "assignee = currentUser()" --paginate
```

### Find tickets in a specific status

```bash
acli jira workitem search --jql 'status = "In Progress"'
acli jira workitem search --jql 'status IN ("To Do", "In Progress")'
```

### Find tickets by project

```bash
acli jira workitem search --jql "project = PROJECTKEY"
```

### Find unassigned tickets

```bash
acli jira workitem search --jql "assignee = EMPTY"
```

### Find tickets by reporter

```bash
acli jira workitem search --jql "reporter = currentUser()"
```

### Find recently updated tickets

```bash
acli jira workitem search --jql "updated >= -7d ORDER BY updated DESC"
```

### Find open bugs in a project

```bash
acli jira workitem search --jql "project = PROJECTKEY AND type = Bug AND status != Closed"
```

## Search & Query Commands

### Search with different output formats

**Table output (default):**

```bash
acli jira workitem search --jql "assignee = currentUser()"
```

**CSV output:**

```bash
acli jira workitem search --jql "project = TEAM" --fields "key,summary,assignee,status" --csv
```

**JSON output:**

```bash
acli jira workitem search --jql "project = TEAM" --json
```

**Count results:**

```bash
acli jira workitem search --jql "assignee = currentUser()" --count
```

### Limit and paginate results

```bash
# Get first 50 results
acli jira workitem search --jql "project = TEAM" --limit 50

# Get all results (paginate through them)
acli jira workitem search --jql "project = TEAM" --paginate
```

### Customize fields

```bash
acli jira workitem search --jql "assignee = currentUser()" --fields "key,summary,priority,status,created"
```

Default fields: `issuetype,key,assignee,priority,status,summary`

### Search using saved filter

```bash
acli jira workitem search --filter 10001
```

### Open search results in browser

```bash
acli jira workitem search --jql "assignee = currentUser()" --web
```

## Work Item Operations

### View a ticket

```bash
acli jira workitem view PROJECTKEY-123
```

### Create a new ticket

```bash
acli jira workitem create \
  --project PROJECTKEY \
  --type Task \
  --title "My new task" \
  --description "Task description"
```

### Edit a ticket

```bash
acli jira workitem edit PROJECTKEY-123 --summary "Updated title"
```

### Assign a ticket

```bash
acli jira workitem assign PROJECTKEY-123 --assignee user@example.com
```

### Transition a ticket (change status)

```bash
acli jira workitem transition PROJECTKEY-123 --transition "In Progress"
```

### Add a comment

```bash
acli jira workitem comment-create PROJECTKEY-123 --comment "This is my comment"
```

### Delete a ticket

```bash
acli jira workitem delete PROJECTKEY-123
```

## Board & Sprint Commands

### List all sprints for a board

```bash
acli jira board list-sprints --board BOARDKEY
```

### List work items in a sprint

```bash
acli jira sprint list-workitems --sprint SPRINTID
```

### Search boards

```bash
acli jira board search --name "My Board"
```

## Real-World Examples

### Get all your open tasks

```bash
acli jira workitem search --jql "assignee = currentUser() AND status != Closed" --paginate
```

### Export team tickets to CSV

```bash
acli jira workitem search --jql "project = LSH AND updated >= -30d" --fields "key,summary,assignee,status,priority" --csv > team_tickets.csv
```

### Find stale tickets (not updated in 60 days)

```bash
acli jira workitem search --jql "updated < -60d AND status != Closed" --paginate
```

### Get all bugs in current sprint

```bash
acli jira workitem search --jql "sprint = OPEN AND type = Bug" --paginate
```

### Find high-priority tickets assigned to you

```bash
acli jira workitem search --jql "assignee = currentUser() AND priority = High" --paginate
```

## Tips & Notes

- **JQL Documentation:** https://www.atlassian.com/software/jira/guides/expand-jira/jira-query-language
- Use `--paginate` when you expect many results (avoids truncation)
- Use `--json` for programmatic processing
- Use `--csv` for spreadsheet export
- Add `ORDER BY updated DESC` to sort by most recent changes
- Use `--web` to open results in your browser instead of the terminal
- Combine multiple conditions: `AND`, `OR`, `NOT`
