---
name: start-ticket
description: "Fetch a ticket from any ticketing system and create an isolated git worktree for it. Use this skill when the user wants to start working on a ticket, begin a new task from their issue tracker, or set up an isolated environment for a ticket. Triggers on phrases like 'start ticket', 'work on MND-1234', 'pick up PROJ-456', 'start working on issue #42', or any mention of starting work on a tracked issue."
---

# Start Ticket

Fetch a ticket from any configured ticketing system, create an isolated git worktree, and prepare the dev environment — all in one step.

## Input

The user provides:
- **Ticket ID** (required): e.g., `MND-1234`, `PROJ-456`, `#42`, or a full URL
- **FE port** (optional): port for the frontend dev server (default: auto-detect an available port)
- **BE port** (optional): port for the backend server (default: use project's default)

Example invocations:
```
/start-ticket MND-1234
/start-ticket MND-1234 4201
/start-ticket MND-1234 --fe-port 4201 --be-port 8002
/start-ticket https://mycompany.atlassian.net/browse/MND-1234
```

## Workflow

### Step 1: Detect the ticketing system

Scan the available MCP tools to determine what ticketing system the user has configured. Look for patterns in the tool names:

| Tool pattern | System |
|---|---|
| `mcp__*atlassian*`, `mcp__*Atlassian*` | Jira (Atlassian) |
| `mcp__*linear*`, `mcp__*Linear*` | Linear |
| `mcp__*github*` or built-in GitHub CLI (`gh`) | GitHub Issues |
| `mcp__*gitlab*` | GitLab Issues |
| `mcp__*shortcut*` | Shortcut (formerly Clubhouse) |
| `mcp__*azure*devops*` | Azure DevOps |

If no ticketing MCP is detected, ask the user to paste the ticket details manually (title, description, acceptance criteria).

### Step 2: Fetch the ticket

Using the detected system, fetch the ticket details. Extract:
- **Ticket ID** (e.g., `MND-1234`)
- **Title/Summary**
- **Description** (keep it concise — first 500 chars if very long)
- **Status**
- **Assignee** (if available)
- **Acceptance criteria** (if available)

Present a brief summary to the user:
```
Ticket: MND-1234 — Implement return order form
Status: To Do | Assignee: Abdalla
---
<short description>
```

### Step 3: Derive the branch name

Convert the ticket into a branch name following common conventions:
- Format: `<ticket-id>-<slugified-title>`
- Lowercase, kebab-case
- Max 60 characters (truncate the title slug, never the ticket ID)
- Strip special characters

Examples:
- `MND-1234` + "Implement Return Order Form" → `MND-1234-implement-return-order-form`
- `PROJ-56` + "Fix login bug on mobile Safari" → `PROJ-56-fix-login-bug-on-mobile-safari`

Present the branch name and ask the user to confirm or modify it before proceeding.

### Step 4: Determine the port

For the frontend dev server port:

1. If the user provided a port, use it
2. Otherwise, try to auto-detect a free port:
   - Check if common ports are in use: `lsof -i :<port> 2>/dev/null`
   - Start from the project's default port (usually 4200) and increment until a free one is found
   - Common range: 4200-4299

For the backend port:
1. If the user provided one, use it
2. Otherwise, use the project's default (check `start.sh`, `proxy.conf.js`, or similar config files)

### Step 5: Create the worktree

Check if the project has a worktree creation script (common names: `create-worktree.sh`, `worktree.sh`, `scripts/worktree.sh`):

**If a script exists:**
Run it with the derived branch name and port. The script likely handles copying config files, installing dependencies, and setting up the environment.

```bash
./create-worktree.sh <branch-name> <fe-port>
```

**If no script exists:**
Create the worktree manually:

```bash
# Create the worktree
git worktree add ../<repo-name>-<port>-<branch-name> -b <branch-name>

# Copy common local config files that are typically gitignored
# (check what exists and copy accordingly — .env, local configs, etc.)
```

If `npm install` or equivalent is needed (check for `node_modules` in worktree), run it.

### Step 6: Present the result

After successful creation, display:

```
Worktree ready!

  Location:   /path/to/worktree
  Branch:     MND-1234-implement-return-order-form
  FE Port:    4201
  BE Port:    8001

Ticket summary:
  MND-1234 — Implement return order form
  <1-2 line description>

Next steps:
  1. Open a new Claude Code session in the worktree directory
  2. Start the dev server: ./start.sh [be-port]
  3. Begin working on the ticket (e.g., /opsx:new or your preferred workflow)

To remove later:
  git worktree remove /path/to/worktree
```

## Error handling

- **Branch already exists**: Ask the user if they want to use the existing branch or pick a different name
- **Worktree already exists at path**: Inform the user and ask how to proceed
- **Ticket not found**: Double-check the ID format, suggest alternatives if the API returns similar results
- **No ticketing MCP available**: Fall back to manual input — ask the user to provide the ticket title and description

## Important notes

- This skill creates infrastructure (worktree, branch) — always confirm the branch name with the user before creating
- Do not start the dev server — the user will do that in their new session
- Do not begin any implementation work — this skill only sets up the environment
- If the user seems to want to continue working in the current session (not create a new one), mention that worktrees are designed for parallel isolated sessions
