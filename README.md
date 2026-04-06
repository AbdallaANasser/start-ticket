# start-ticket

A [Claude Code](https://claude.ai/code) plugin that fetches a ticket from **any** ticketing system, creates an isolated git worktree, and configures your dev environment — all in one command.

## What it does

```
/start-ticket MND-1234 4201
```

1. **Detects your ticketing system** — scans your configured MCP tools (Jira, Linear, GitHub Issues, GitLab, Azure DevOps, Shortcut) and fetches the ticket automatically
2. **Creates an isolated worktree** — new git worktree with a branch derived from the ticket ID and title
3. **Configures dev server ports** — sets up your frontend (and optionally backend) port so you can run multiple tickets in parallel
4. **Prints next steps** — tells you exactly where to go and what to run

No ticketing MCP configured? No problem — it falls back to asking you to paste the ticket details.

## Why worktrees?

Worktrees let you work on multiple tickets simultaneously without stashing, switching branches, or rebuilding `node_modules`. Each ticket gets its own directory, its own branch, and its own dev server on a unique port.

```
~/projects/
  my-app/                    # main branch — port 4200
  my-app-4201-MND-1234-.../  # ticket 1 — port 4201
  my-app-4202-MND-5678-.../  # ticket 2 — port 4202
```

## Installation

### Option 1: Add as a plugin marketplace (recommended)

```bash
/plugin marketplace add https://github.com/AbdallaANasser/start-ticket
/plugin install start-ticket@agentic-dev-tools
```

### Option 2: Via `settings.json`

Add to your `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "agentic-dev-tools": {
      "source": {
        "source": "github",
        "repo": "AbdallaANasser/start-ticket"
      }
    }
  },
  "enabledPlugins": {
    "start-ticket@agentic-dev-tools": true
  }
}
```

### Option 3: Copy into your project

Copy the `skills/start-ticket/` directory into your project's `.claude/skills/`:

```bash
cp -r skills/start-ticket /path/to/your/project/.claude/skills/
```

## Usage

```
/start-ticket <ticket-id> [fe-port] [--be-port <port>]
```

### Examples

```bash
# Basic — auto-detect port
/start-ticket MND-1234

# With specific frontend port
/start-ticket MND-1234 4201

# With both FE and BE ports
/start-ticket MND-1234 4201 --be-port 8002

# From a full URL
/start-ticket https://mycompany.atlassian.net/browse/MND-1234
```

## Supported ticketing systems

The skill auto-detects your ticketing system from configured MCP tools:

| System | Detected via |
|---|---|
| Jira | `mcp__*atlassian*` / `mcp__*Atlassian*` tools |
| Linear | `mcp__*linear*` / `mcp__*Linear*` tools |
| GitHub Issues | `mcp__*github*` tools or `gh` CLI |
| GitLab | `mcp__*gitlab*` tools |
| Azure DevOps | `mcp__*azure*devops*` tools |
| Shortcut | `mcp__*shortcut*` tools |
| None | Falls back to manual input |

## What happens under the hood

1. **Ticket fetch** — reads ticket ID, title, description, status
2. **Branch naming** — generates `<TICKET-ID>-<slugified-title>` (confirms with you before creating)
3. **Worktree creation** — if your project has a `create-worktree.sh`, it uses that (so your custom setup is preserved). Otherwise, creates a worktree manually
4. **Port detection** — if no port specified, finds the next available port starting from your project's default
5. **Summary** — prints ticket summary, worktree location, and next steps

## Works great with

- **OpenSpec** (`/opsx:new`) — after creating the worktree, start a new session and begin spec-driven development
- **Any CI/CD workflow** — each worktree is a full git checkout, so your existing tools work as-is
- **Team conventions** — the skill respects your project's existing worktree scripts and config

## Project-specific worktree scripts

If your project has a `create-worktree.sh` (or similar), the skill will use it instead of creating the worktree manually. This means your project-specific setup (copying local configs, installing dependencies, configuring MCP servers, IDE setup) is automatically preserved.

## License

MIT
