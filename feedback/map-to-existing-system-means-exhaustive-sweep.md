---
name: map-to-existing-system-means-exhaustive-sweep
description: In structured requirement/impact-analysis prompts, "map to existing system" means an exhaustive grep-driven sweep for every consumer of the touched entity/field, not just the modules the ticket happens to name
metadata:
  type: feedback
---

When running a structured requirement/impact analysis (business goal → system map → gap analysis →
impact analysis → design → tasks → edge cases → stakeholder questions), the "map to existing
system" / "files/modules that will change" / "risk areas" steps must be **ticket-agnostic** from
the start: grep across the whole codebase for every reader/writer of the entity or field being
changed, not just the screens/services the ticket text happens to mention.

**Why:** on AUT-3639 (carplanet booking→multi-vehicle ticket), the first pass mapped only the 7
modules the client's ticket named (Booking Screen, Sales Diary, Vehicle/
Stock, Internal Transport, Pricing, ETA Tracker, Reporting) and called Step 4's "audit all direct
reads of booking.vehicle_id" a to-do rather than doing it. User pushed back: a client-written ticket
only describes user-visible touchpoints they know about — it has no reason to mention
`docusign-html-templates.ts`, a third-party integration (Audiofy), SMS content, or a dedicated
`booking_reservation_log` table, all of which read/write the same field. A second grep-first sweep
found ~13 more backend services and ~6 more frontend views outside the ticket's own list, including
customer-facing legal PDFs and an external integration — a materially different risk/size picture.
The prompt structure itself was correct and demanded this; the miss was letting the ticket's own
vocabulary narrow what got searched for.

**How to apply:** whenever a requirement-analysis prompt includes a "map to existing system" /
"impact analysis" step, do the full grep-for-every-consumer-of-the-touched-field sweep as part of
that step's first pass — don't scope it to ticket-named modules and don't defer it to "flagged for
follow-up." Use a subagent for the sweep (breadth search, not depth) per [[token-economy]]. This
applies to any ticket describing a schema/entity-level change, not just AUT-3639.
