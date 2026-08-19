---
name: session-start-skills-hook
description: "The SessionStart hook that auto-pulls the claude-skills repo at every Claude Code launch — where it lives, what it does, and how to restore it on a new machine"
metadata:
  node_type: memory
  type: reference
---

A **Claude Code `SessionStart` hook** keeps the claude-skills repo current automatically at every launch,
so I'm always on the latest skills without being reminded (the bulletproof version of [[session-start-routine]]
step 2 / [[claude-skills-repo]] — memory alone only reminds; this hook actually runs).

**Where it lives (this machine):**
- Script: `~/.claude/hooks/pull-skills.sh` (executable).
- Wired in `~/.claude/settings.json` under `hooks.SessionStart` → a `command` hook pointing at that script,
  ALONGSIDE the existing `PreToolUse` token-gate hook (don't clobber that when editing).

**What it does:** `git fetch` the repo, then **fast-forward pull only when behind AND the tree is clean**;
if dirty + behind it warns and does NOT touch local changes; if diverged it flags for manual resolve; if
current it stays silent. Always `exit 0` so a git/network hiccup never blocks the session.

**Takes effect next launch** (the firing session's SessionStart has already passed). Updated existing
skills apply immediately on pull; a brand-new skill folder only auto-registers at the following launch.

**Backed-up copy:** the canonical script is committed in this repo at `scripts/pull-skills.sh`. The live
hook at `~/.claude/hooks/` is NOT under the memory repo, so this copy is the restore source.

**Restore on a new machine:**
1. `cp <memory-repo>/scripts/pull-skills.sh ~/.claude/hooks/pull-skills.sh && chmod +x ~/.claude/hooks/pull-skills.sh`
2. Add to `~/.claude/settings.json` under `hooks.SessionStart` a command hook with
   `"command": "/Users/<you>/.claude/hooks/pull-skills.sh"` (fix the path), keeping any existing hooks.
3. Fix the `REPO=` path inside the script if the claude-skills clone isn't at the same location.
4. `jq -e '.hooks | keys' ~/.claude/settings.json` to confirm the JSON is still valid.
