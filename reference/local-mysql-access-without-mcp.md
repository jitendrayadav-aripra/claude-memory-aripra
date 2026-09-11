---
name: local-mysql-access-without-mcp
description: no local MySQL MCP server is configured on this machine — connect directly via mysql2 + car-planet-backend/.env instead, and mind the CRLF .env-parsing gotcha
metadata:
  type: reference
---

`car-planet-backend/CLAUDE.md` documents a local read-only MySQL MCP server (`.mcp.json`,
`mcp_server_mysql`) for inspecting the live schema/data. **That file is gitignored and does not
exist on this machine** — confirmed 2026-09-11 (`cat .mcp.json` → not found; `.gitignore:57` lists
it). So no local-DB tool is ever surfaced this session, unlike `mysql_prod` (a separate,
already-configured connector pointed at **production** — do not use it for local verification
without explicitly confirming that's what the user wants, per
[[verify-claims-before-posting]]-style caution around prod data).

**What actually works instead — connect directly, no MCP needed:**
1. `mysql2` is already available as a transitive dependency in
   `car-planet-backend/node_modules/mysql2` (via TypeORM) — no install required.
2. Read `car-planet-backend/.env` for `DB_HOST`/`DB_USER`/`DB_PASSWORD`/`DB_NAME`/`DB_PORT`.
3. Write a small script to the session scratchpad, `require("D:/aripra/.../mysql2/promise")` (use
   plain `D:/...` forward-slash paths — POSIX-style `/d/...` paths from Git Bash do NOT resolve in
   a native Windows Node `require()`), connect, run the read-only query, print results.
4. This exact approach (no MCP tool at all) was also how local DB verification was done in an
   earlier session (see AG-260's history) — it's the established fallback, not a workaround.

**Gotcha that cost real debugging time (2026-09-11):** `car-planet-backend/.env` has **CRLF** line
endings. A naive parser like `envText.split("\n").forEach(line => line.match(/^(...)=(.*)$/))`
silently matches **zero** lines — JS's `.` never matches `\r`, so the trailing `\r` left on every
line (after splitting only on `\n`) blocks `$` from ever reaching true end-of-string, and the whole
regex fails rather than just corrupting the value. Fix: `envText.split(/\r?\n/)` instead of
`split("\n")`.

**Expect an auto-mode classifier prompt** the first time a script makes a live DB connection (even
read-only) — it gets flagged as sensitive; explain what/why and wait for explicit approval rather
than retrying or working around it.

See [[issue-AG-287-overdue-fitting-instock-statuses]] for the full worked example (script,
symptoms, fix, and the actual verification queries run).
