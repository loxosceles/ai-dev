---
name: mcp-server-setup
description: How to configure MCP servers for Kiro, VS Code, and Claude in devcontainers. Follow when adding, debugging, or migrating MCP server configurations.
type: guideline
---

# MCP Server Setup

**This is a strict guideline.** Follow these rules exactly.

MCP server configuration in our devcontainer setup. Covers config file locations, format differences between agents, common pitfalls, and the install/test workflow.

---

## Config File Locations

Each agent reads MCP config from a different path. In our devcontainer setup, Kiro's config lives on the host (mounted into the container).

| Agent | Config path (container) | Host source |
|-------|------------------------|-------------|
| Kiro CLI | `~/.kiro/settings/mcp.json` | `~/.devcontainer-state/cache/{project}/kiro/settings/mcp.json` |
| VS Code | `.vscode/mcp.json` (in project) | Committed to repo |
| Claude Code | `~/.claude/settings.json` | `~/.devcontainer-state/cache/{project}/claude/settings.json` |

Kiro's config is per-project because it's mounted from `cache/{project}/kiro/`.

---

## Config Formats

### Kiro CLI

Top-level key: `mcpServers`

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "package-name"],
      "env": {
        "ENV_VAR": "value"
      }
    }
  }
}
```

### VS Code

Top-level key: `servers`. Supports `inputs` for interactive prompts (useful outside devcontainers).

```json
{
  "servers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "package-name"],
      "env": {
        "ENV_VAR": "${ENV_VAR}"
      }
    }
  }
}
```

### Claude Code

MCP servers are nested inside the settings file under `mcpServers`:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "package-name"],
      "env": {
        "ENV_VAR": "value"
      }
    }
  }
}
```

---

## Known Servers and Their Env Vars

Use the exact env var names listed here. Mismatched names are the most common setup failure.

### Trello (`mcp-server-trello`)

```
TRELLO_API_KEY    — from https://trello.com/app-key
TRELLO_TOKEN      — NOT "TRELLO_API_TOKEN"
TRELLO_BOARD_ID   — optional, found in board URL
```

### GitHub (`@modelcontextprotocol/server-github`)

```
GITHUB_PERSONAL_ACCESS_TOKEN  — or GITHUB_TOKEN
```

---

## Adding a New MCP Server

### 1. Install the package globally in the container

```bash
npm install -g <package-name>
```

This uses the user-local npm prefix (`~/.npm-global/`) so the `vscode` user has write access.

### 2. Test it responds to MCP initialize

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}' \
  | ENV_VAR=value timeout 5 npx <package-name> 2>/dev/null
```

Expected: a JSON response with `result.serverInfo`. If you get an error about missing env vars, the var names are wrong — check the server's source or README.

### 3. Write the config

Edit the appropriate config file for the agent (see Config File Locations above). Use the exact env var names from the test that worked.

### 4. Restart the agent session

MCP configs are read at session start. Restart Kiro/Claude/VS Code to pick up changes.

---

## Devcontainer Considerations

- **Global npm packages persist** across container restarts (mounted via `~/.cache`) but are lost on rebuild. For permanent installs, add to `Dockerfile` or `post_create.sh`.
- **Kiro config is on the host.** Edit `~/.devcontainer-state/cache/{project}/kiro/settings/mcp.json` from the host, or `~/.kiro/settings/mcp.json` from inside the container — they're the same file.
- **Credentials in MCP configs are gitignored** because Kiro and Claude configs live in `cache/` (gitignored in devcontainer-state). VS Code's `.vscode/mcp.json` is committed — use `${ENV_VAR}` references there, not raw values.

---

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Server not found | Package not installed | `npm install -g <package>` |
| "missing env var" error | Wrong env var name | Check Known Servers section; test manually |
| Server starts but agent can't connect | Config in wrong file or wrong format | Check `mcpServers` vs `servers` key |
| Works locally, fails in container | Config on host not mounted | Verify `~/.devcontainer-state/cache/{project}/kiro/settings/mcp.json` exists |
| Agent doesn't see config changes | Stale session | Restart the agent session |

---

## Progressive Improvement

If the developer corrects a behavior that this skill should have prevented, suggest a specific amendment to this skill to prevent the same correction in the future.
