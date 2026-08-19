---
name: cross-reference-linked-tickets
description: "When a GitLab ticket references another ticket/commit that's already in memory, auto-pull that memory — don't wait to be reminded"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 949bc40a-0319-4fca-a1a5-306f086b798d
---

When a GitLab ticket (description, comment, or thread) references **another ticket, commit, or MR**
that is ALREADY tracked in our memory index, automatically consider that linked memory as context —
read its NOW block, related commit refs, and decisions BEFORE acting. Do not wait for the user to
remind you; they explicitly will not.

**Why:** the linked ticket usually carries the root cause, the prior fix/commit, the convention, or a
decision that directly shapes the current work. Pulling it from memory (instead of re-deriving from
the codebase or re-reading big files) is faster and far cheaper on tokens — same payoff as checking an
old commit ref before re-implementing a similar change (see [[check-recent-commit-before-tweaking]]).

**How to apply:**
- On opening any ticket, scan it for `#<num>` ticket refs, commit SHAs, and MR links. For each, check
  MEMORY.md — if it's tracked, load that memory's NOW block first.
- Treat a tracked lineage/sibling (e.g. the [[asset-url-removal-shared-event]] chain, or a
  "port of #X" / "same class as #X" note) as a signal to read that entry, not just mention it.
- Fold what you learn into the current reply/fix; cite the linked ticket/commit so the trail is clear.
- If the linked ticket is NOT in memory, a quick `glab` fetch of just its title/last note is fine —
  still cheaper than blind code reading.

Pairs with [[token-economy]] and [[session-start-routine]].
