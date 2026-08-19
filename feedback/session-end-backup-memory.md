---
name: session-end-backup-memory
description: "At session end (or when the user wraps up / says 'back up memory'), commit + push the memory repo so the GitHub backup stays current"
metadata:
  type: feedback
---

The memory store `~/Desktop/Projects/MikeStuff/memory/` is a git repo backed up to the PRIVATE GitHub repo
**`ppogra23/claude-memory`** (personal account, owner-only — NOT under any org). Memory edits I make during a
session (worklog, ticket NOW-block updates, new rules) are local-only until committed, so the backup goes stale.

**Routine — when wrapping up a session, or when the user says "back up memory" / "done":** if the memory repo has
uncommitted changes, commit them and push:
```
cd ~/Desktop/Projects/MikeStuff/memory && git add -A && git commit -m "<short summary of memory changes>" && git push
```

**Why:** user asked (2026-06-22) to make backing up the memory part of the session-end routine so the GitHub copy
stays current. **Privacy is mandatory:** the repo holds internal data (user emails, support cases, event IDs) — it
MUST stay private and personal-only; never add org/team access, never move it to `lumasoft-co` or `ariprainfotech`.

**How to apply:** one commit per session is fine (summarize what changed — tickets touched, rules added). Don't
push mid-session on every edit (noise). New machine? `setup.sh` in the repo re-wires the global `~/.claude/CLAUDE.md`
+ symlinks after cloning. Companion to [[session-start-routine]] and [[commit-discipline]]
(the memory-repo backup commit is routine maintenance — no need to ask first, unlike code commits).
