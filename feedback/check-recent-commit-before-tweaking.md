---
name: check-recent-commit-before-tweaking
description: "Before tweaking a change, read the recent commit/uncommitted diff that introduced it — don't re-read big files blind"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 949bc40a-0319-4fca-a1a5-306f086b798d
---

When a new ask is a tweak/fix on top of recent work (UI adjustment, follow-up on a feature just built, change to "the same thing" a recent commit touched), FIRST read that source of truth — the relevant recent commit (`git show <ref>`) or the uncommitted working-tree diff (`git diff`) — to learn exactly what was done and what conventions/values it used. Only then plan the tweak.

**Why:** the diff/commit is small and tells you precisely what to change and the established pattern to match; re-reading the whole large file (e.g. a 2,500-line storyboard) to rediscover it burns context cumulatively across the turn's tool round-trips and is what eats tokens. Memory stores decisions, not file contents, so the diff is the cheap bridge.

**How to apply:** on any "tweak/adjust/fix what we just did" request, `git diff` (uncommitted) or `git show`/`git log -p` the related commit ref before opening the target file. RECORD every commit ref in the ticket memory NOW block as soon as it's made (short hash + one-line what-it-changed) — that's what makes the next tweak a one-lookup `git show` instead of a big-file re-read. Extends [[reuse-existing-code]] (reuse over reinvent).
