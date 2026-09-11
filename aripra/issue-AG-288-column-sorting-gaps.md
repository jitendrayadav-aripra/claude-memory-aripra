---
name: issue-AG-288-column-sorting-gaps
description: AG-288 — add missing column-header sorting across 6 Parts Inventory tables/panels; fixed a real MRT icon-placement quirk and a TanStack sortability gap along the way
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-11.** `tsc` clean both repos, `next lint` clean on every changed
frontend file. Closed by explicit instruction. No migration, ~10 files across both repos.

**Core ask:** several columns across Parts Inventory were missing sorting, and 2 views (People
pivot, Overview drill-down panel) had none at all. Reference pattern reused throughout:
`non_conforming_tab.tsx`'s material-react-table + `manualSorting: true` + per-column backend
`sortMap` → `.orderBy()`.

**Built, 6 areas:**
1. **Alert Checks Late/Breach/Gone** — 6 columns flipped sortable
   (status/location/part/supplier/poBy/technician); extended 3 backend `sortMap`s.
   `poBy`/`technician` sort via `CONCAT(firstName, ' ', lastName)` (split-name columns, no
   single-column precedent to reuse).
2. **Ready Cars** — location/workshopManager flipped sortable; backend `sortMap` extended.
3. **Consumables stock list** — frontend-only; backend already sorted generically by `sort.id`
   matched against the row object.
4. **People detail list** — vehicleStatus/location/part/supplier flipped sortable; backend
   `sortMap` extended (reuses `ALERT_ROW_SELECT`, same column sources as #1).
5. **People pivot** — plain `<table>` → MRT, **client-side** sort (deliberate deviation from the
   ticket's suggested backend param, confirmed with the user first): this endpoint already returns
   every row unbounded in one shot, so a server round-trip for sorting adds no correctness benefit.
6. **Overview drill-down panel** — plain `<table>` → MRT, **server-side** sort (unlike #5, this one
   is capped at 200 rows, so client-side sorting over that fixed slice would misrepresent the true
   top/bottom for any field other than the default). Preserved AG-284's conditional `extraColumns`.

**Two real bugs found and fixed after the user checked the actual UI (not caught by `tsc`/lint,
since both are runtime/library-behavior issues):**

- **Bug A — sort icon appeared BEFORE the column name** on every right-aligned numeric/count
  column (a hard requirement the user stated at the very start of this ticket). Traced into MRT's
  own installed source (`material-react-table/dist/index.esm.js`): the header's label+icon wrapper
  does `flexDirection: tableCellProps?.align === 'right' ? 'row-reverse' : 'row'` — right-aligning
  the **head** cell is what MRT ties the icon-reversal to. First fix attempt (remove
  `muiTableHeadCellProps` align, keep body right-aligned) caused a NEW visible bug — header left,
  value right, disconnected in a wide column (user caught via screenshot). Final fix:
  **left-align both head and body** on every affected numeric column across all 6 areas — the only
  combination that satisfies "icon after name" AND keeps header/value visually stacked together.
  This also silently fixed the identical pre-existing issue on Alert Checks' Days/£ and Ready
  Cars' Unfitted parts/Longest wait/£ on shelf, which predated this ticket.
- **Bug B — Parts Arrived Date / First Flagged Date / Supplier Window / Return Days Left had no
  sorting at all** despite correct backend `sortMap` entries. Traced into TanStack Table's core
  (`@tanstack/table-core`): `column.getCanSort()` hard-requires `!!column.accessorFn` — these 4
  "extra" columns were built with only `id` + `Cell` (no accessor), so sorting was structurally
  impossible regardless of `enableSorting`. Fixed by giving each a real `accessorFn` returning its
  underlying raw value (unused for the actual sort, which is server-side — just needs to exist).

**Related:** [[issue-AG-284-drilldown-return-window-columns]] (the extra columns this ticket made
sortable), [[issue-AG-258-people-pivot-split-90d-no-date]] (the No Date/90D+ columns in the pivot).

---

## HISTORY

- 2026-09-11: Ticket opened by user request ("new ticket is AG-288"), memory file created
  immediately per [[create-ticket-file-immediately-on-open]] before any analysis. User's opening
  note ("sort icon after the column name, not before") seemed like it'd be automatically satisfied
  by MRT's defaults — turned out to be the single hardest part of this ticket.
- 2026-09-11: Fetched from Jira, verified all 6 areas against current code (line numbers had
  drifted from AG-285/286/287/289). Flagged the People-pivot client-vs-server-side sorting
  decision explicitly rather than silently picking one. User approved the full plan. Built all 6
  areas across ~10 files. `tsc`+`next lint` clean.
- 2026-09-11: User checked the actual UI and reported 5 groups of real issues: icon-before-name on
  Consumables/People-pivot/3 drilldown-panel tables, and zero sorting at all on the drilldown
  panel's 4 extra columns. Traced both root causes into MRT's and TanStack's own installed source
  rather than guessing (see NOW) — confirmed both are real, structural library-behavior gaps, not
  typos. Explained findings, proposed fixes, asked permission.
- 2026-09-11: Built both fixes. User immediately caught (via screenshot) that removing only the
  head-cell right-align created a NEW visible bug — header and value now visually disconnected in
  wide columns. Explained the tension (can't right-align only one side without either the icon
  flipping or the values disconnecting) and proposed left-aligning both sides consistently. User
  approved; applied across all 6 files, including quietly fixing the same pre-existing issue on
  Alert Checks/Ready Cars columns that predated this ticket. `tsc`+`next lint` clean again.
- 2026-09-11: User asked for overall commit messages (nothing committed yet across the whole
  ticket), then said to mark AG-288 done.
