---
name: session-end-backup-memory
description: "At session end (or when the user wraps up / says 'back up memory' / runs /backup), commit + push the memory repo so the GitHub backup stays current"
metadata:
  type: feedback
---

The memory store `D:\aripra\projects\claude-memory-core-store` (bash: `/d/aripra/projects/claude-memory-core-store`)
is a git repo backed up to the PRIVATE GitHub repo **`jitendrayadav-aripra/claude-memory-aripra`**
(`git@github.com:jitendrayadav-aripra/claude-memory-aripra.git`, branch `master`). Memory edits I make during a session
(worklog, ticket NOW-block updates, new rules) are local-only until committed and pushed, so the backup goes stale.

**Routine: when wrapping up a session, or when the user says "back up memory" / "done" / runs `/backup`:**
```
cd /d/aripra/projects/claude-memory-core-store && git remote -v && git add -A && git status --short \
  && git commit -m "<short summary of memory changes>" && git push origin master
```
- Check `git remote -v` first. If it isn't `jitendrayadav-aripra/claude-memory-aripra`, stop and ask.
- Read any untracked file I didn't create this session before committing it.
- If nothing changed and nothing is unpushed, say so; no empty commit.

**Why:** backing up memory is part of the session-end routine, so the GitHub copy stays current. **Privacy is
mandatory:** the repo holds internal data (client tickets, internal Jira details, user emails), so it MUST stay
private and owner-only. Never add team/org access, and never move it under an organisation.

**History:** until 2026-10-01 this file (and `~/.claude/commands/backup.md`) carried a different person's setup:
the path `~/Desktop/Projects/MikeStuff/memory` and the remote `ppogra23/claude-memory`. That mismatch made the
permission check block `/backup` pushes twice (2026-09-25 and 2026-09-30). Both files now name the real repo.

**How to apply:** one commit per session is fine (summarize what changed: tickets touched, rules added). Don't
push mid-session on every edit (noise). Companion to [[session-start-routine]] and [[commit-discipline]] (the
memory-repo backup commit is routine maintenance, so no need to ask first, unlike code commits).
