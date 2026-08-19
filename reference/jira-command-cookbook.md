---
name: jira-command-cookbook
description: "Jira ops cookbook (via the Atlassian Rovo MCP connector) for this project."
metadata: { node_type: memory, type: reference }
---
Project key: `PROJ`. Auth: authenticate the **Atlassian Rovo** connector INSIDE Claude Code
(`mcp__claude_ai_Atlassian_Rovo__authenticate` → complete the returned URL) — not a shell command.
- Read/search/comment issues via the Atlassian MCP tools.
- Ticket files keyed by issue key: `aripra/issue-<KEY>-*.md`.
- Smart commits: include `<KEY>` in the message (e.g. `PROJ-123 #comment …`).
