---
name: aut3639-update-four-docs-every-change
description: For AUT-3639 (multi-vehicle booking), every code or decision change must update the four docs in my-docs/projects/Booking-AUT-3639-5th-project/ in the same turn
metadata:
  type: feedback
---

Every change on AUT-3639 (multiple vehicles on one booking), whether code, a client answer, or a
design decision, must update all four docs in
`my-docs/projects/Booking-AUT-3639-5th-project/` in the same turn:

1. `BOOKING_LINKING_MULTIPLT_VEHICLE_WITH_SINGLE_BOOKING_3639.md`: the living feature reference.
   Keep status, design decisions, endpoints, UI, files changed, deferred items and the test
   checklist current.
2. `project-log-whatimptablesormigration-added.md`: backend change log, newest first. Add a dated
   entry for any table, column, migration, endpoint or backend behaviour change, with a short
   explanation of why.
3. `date-wise-daily-update.md`: point-wise daily updates in plain language for the senior /
   client (no code terms). Add or extend today's dated section.
4. `questions.md`: add every new open question the change raises. When a question is answered,
   move it to "Resolved" with the date, who answered it, and what was decided.

**Why:** the user asked for this explicitly (2026-09-30). These four files are how they report
to the senior and the client, and how anyone later answers "what changed and why".
**How to apply:** treat it as a mandatory closing checklist for any AUT-3639 turn that changed code
or recorded a decision, alongside [[always-provide-commit-message-after-changes]].
