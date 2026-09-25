---
name: dont-conflate-harness-memory-with-this-store
description: the harness's own built-in MEMORY.md (auto-loaded, different path) is not this store's MEMORY.md — always explicitly load claude-memory-core-store/MEMORY.md at session start, never skip it because "a MEMORY.md was already loaded"
metadata:
  type: feedback
---

Claude Code's harness has its own built-in automatic memory system with its own `MEMORY.md`,
auto-loaded at `C:\Users\jiten\.claude\projects\<project-hash>\memory\` before any of my own
actions run. This is a **different file** from this store's `MEMORY.md`
(`/d/aripra/projects/claude-memory-core-store/MEMORY.md`), despite the identical filename.

The global CLAUDE.md instruction says: "if MEMORY.md has not already been loaded, read
`.../claude-memory-core-store/MEMORY.md`." On a brand-new-ticket session (no `/rt <id>` resume,
which globs straight into this store and sidesteps the ambiguity), I incorrectly read "a MEMORY.md
is already loaded" (the harness's own) as satisfying that instruction, and never actually loaded
this store's real index. Result: worked an entire ticket (AG-309) without any of this store's
conventions/feedback loaded, including a rule that already existed
([[mark-as-done-means-memory-not-jira]]) — re-asked something already settled and wasted the
user's time confirming it.

**Why:** caught directly by the user, who noticed this session behaved as if it had no memory of
established conventions, while a same-day ticket resumed via `/rt` worked fine.

**How to apply:** at the start of every session, explicitly load
`/d/aripra/projects/claude-memory-core-store/MEMORY.md` regardless of whether the harness's own
built-in `MEMORY.md` appears already loaded in context — they are never the same file. Don't treat
the harness's auto-loaded memory as a substitute for this store under any circumstance.
