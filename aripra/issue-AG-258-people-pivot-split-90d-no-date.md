---
name: issue-AG-258-people-pivot-split-90d-no-date
description: AG-258 — split the People tab pivot's combined "90D+ / no date" column into two distinct columns (aging backlog vs. data-entry gap)
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-09.** `tsc`+`next lint` clean both repos. Closed by explicit
instruction. Part of the Parts Oversight module — documented in `NEW_PARTS_AND_STOCK_INVENTORY.md`
under "Related ticket — AG-258".

Follow-up ticket (originally noted 01 Sep 2026 in Jira, parked as a scoped refinement, picked up
and built 2026-09-09). Small, additive change exactly as scoped in the original ticket text — no
new joins, no new data source, no migration.

- **Backend** (`getPeopleAttributionPivot`, `inventory.service.ts`) — the original single
  `SUM(CASE WHEN partArrivedDate IS NULL OR DATEDIFF(NOW(), partArrivedDate) >= 90 THEN 1 ELSE 0 END)`
  merged two distinct signals into one count. Split into two mutually exclusive SUMs:
  - `over90`: `partArrivedDate IS NOT NULL AND DATEDIFF(NOW(), partArrivedDate) >= 90` — genuinely
    aged parts, a chronic aging-backlog problem.
  - `noDate`: `partArrivedDate IS NULL` — a data-entry gap, age genuinely unknown.
  Same `buildPeopleWaitingBase()` filter and same per-role joins (poBy/technician/mechOwner)
  untouched — only the SELECT/aggregation changed.
- **Frontend** — `PeopleAttributionPivotRow` type (`parts_inventory.api.ts`) gained `noDate: number`.
  `people_tab.tsx`'s pivot table split the single `<th>90D+ / no date</th>` + combined cell into two
  headers ("90D+", "No Date") and two independently red-highlighted cells (same `>0` → red-bold
  styling convention as the original single cell). Checked `people_person_table.tsx` (the detail
  list) first — confirmed it doesn't reference this bucket at all, so no other file needed touching.

**Related:** none directly — this is a standalone refinement of the People tab pivot built during
AUT-3572's Checkpoint 3, not tied to another open ticket.

---

## HISTORY

- 2026-09-09: User opened with the exact parked Jira follow-up text (01 Sep 2026 note) describing
  the problem and the intended fix (split into two SUM(CASE) columns) almost verbatim, asking me to
  explain understanding first. Traced `getPeopleAttributionPivot` (`inventory.service.ts:5988`) and
  its two frontend consumers (`PeopleAttributionPivotRow` type in `parts_inventory.api.ts`,
  `people_tab.tsx`'s pivot table markup) before proposing the plan — confirmed no other file
  (`people_person_table.tsx`) touches this same bucket. Explained understanding + plan, got "yes go
  ahead", implemented directly (no design ambiguity — ticket text already specified the exact SQL
  change). `tsc` clean both repos, `next lint` clean on both changed frontend files.
- 2026-09-09: User asked for commit messages for both repos, then said "mark this ticket AG-258 as
  done" — closed, no further changes.
