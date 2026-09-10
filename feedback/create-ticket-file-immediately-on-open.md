---
name: create-ticket-file-immediately-on-open
description: create the ticket's memory file and add it to MEMORY.md's active index the moment "new ticket: X" is said, not at close-out or when asked
metadata:
  type: feedback
---

The instant the user says "new ticket: <ID>" (or resumes one with no existing file), create
`aripra/issue-<ID>-<slug>.md` with at least a minimal `## NOW` block ("Status: opened, gathering
requirements") and add its line to `MEMORY.md`'s active index — **before** doing any analysis,
before drafting a plan, before writing any code. Update the file's `NOW`/`HISTORY` as the ticket
progresses (plan agreed, built, verified), the same way it's already updated at close-out — it just
starts existing much earlier now.

**Why:** repeated twice in one session (AG-261, then AG-285) — a ticket was fully explained,
approved, and built before any memory file existed for it, and only got created after the user
directly asked "is this tracked as active?" The user correctly called out that the first correction
didn't take: fixing the one instance again wasn't enough, the underlying habit needed to change.
`MEMORY.md`'s own spec already says the index should list every ticket with a pending action on
me — the gap was treating file creation as a close-out step instead of an open step.

**How to apply:** on "new ticket: AG-XXX" / "resume ticket AG-XXX" (when no file exists yet) / any
equivalent opening signal — create the file and index line in the same turn, before responding
with analysis. Don't wait to be asked, and don't wait until the ticket is done to write it down.
