---
name: issue-AG-284-drilldown-return-window-columns
description: AG-284 — First Flagged/Arrival Date, Supplier Window, and Return Days Left columns on the Awaiting Supplier Return and Return Window Unknown Overview drilldowns
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-08.** `tsc`+`next lint` clean both repos. Closed by explicit
instruction. Part of the Parts Oversight module — documented in
`NEW_PARTS_AND_STOCK_INVENTORY.md` under "Related ticket — AG-284".

**Both "Awaiting Supplier Return" and "Return Window Unknown" are the same shared component**
(`parts_inventory_drilldown_panel.tsx`), opened with different `resolutionStage` params
(`WAITING_TO_BE_RETURNED` vs `UNKNOWN`) against the same backend endpoint
(`getPartsInventoryDrilldown`, `inventory.service.ts`). New columns are conditional on
`params.resolutionStage` via a memoized `extraColumns` array, inserted between the existing
Technician and Days columns — every other bucket/resolution-stage is unaffected.

- **Backend** — `getPartsInventoryDrilldown` now also selects `taskPart.firstFlaggedDateText`,
  `taskPart.partArrivedDateText`, `taskPart.partArrivedDate`, `poSupplier.returnWindowDays`. The
  `po`/`poSupplier` join already existed for this bucket (added back in AG-260-era resolution
  logic) — no new join, no migration (all source columns already existed).
- **"Awaiting Supplier Return" (`WAITING_TO_BE_RETURNED`)** — 4 columns, in order: First Flagged
  Date, Parts Arrived Date, Supplier Window, Return Days Left.
  - Both dates go through `formatTableDate()` (established `*DateText` convention).
  - "Supplier Window" shows `"{returnWindowDays} days"` or "—". Originally named "Actual Supplier
    Window" — renamed to just "Supplier Window" per a same-day follow-up request (drop "Actual").
  - "Return Days Left" reuses `computeDaysLeft(partArrivedDate, returnWindowDays)` **imported
    directly** from Part Returns Audit's `non_conforming_tab.tsx` (already exported there) — per
    the user's explicit instruction to use that module as reference. Deliberately did NOT copy
    that module's full 8-state "signed-off" coloring logic, since TaskPart resolution-stage rows
    here don't have that PO-sign-off concept — used a simplified version instead (plain "N days",
    red ≤1 day / orange =2 days / default otherwise, matching the same color thresholds).
- **"Return Window Unknown" (`UNKNOWN`)** — 2 columns, in order: Part Arrival Date, First Flagged
  Date. Note the reversed lead column vs. the other drilldown — this matches the ticket's own
  column-order spec exactly, not an oversight.
- **"Days" column renamed to "Days Since Arrival"** — same-day follow-up. The user noticed "Days"
  (existing column, `DATEDIFF(NOW(), partArrivedDate)`, shared by all 8 buckets this panel serves)
  read as confusingly similar to the new "Return Days Left" and asked for a clearer name or a
  better suggestion; recommended "Days Since Arrival" (Title Case, consistent with the header
  meaning across every bucket, not just these two) — user approved as-is.

**Related:** [[issue-AG-277-overview-label-rewording]] (same drilldown panel, prior UI-copy pass).

---

## HISTORY

- 2026-09-08: User opened with 2 screenshots (Overview + the two drilldown panels) and a ticket
  with no Jira description, listing the exact 4 + 2 columns and pointing at Part Returns Audit's
  Return Window/Return Days Left columns as the reference for the calculation. Traced both
  drilldowns to the single shared `parts_inventory_drilldown_panel.tsx` component and the single
  backend `getPartsInventoryDrilldown` function (already joining `poSupplier` for this bucket but
  not selecting the needed raw fields) before proposing a plan — explained understanding, gave a
  title+description draft, asked permission per standing workflow.
- 2026-09-08: Built after "yes go for it" — backend select+type+mapping additions, frontend
  `extraColumns` memo conditional on `resolutionStage`, reused `computeDaysLeft` import. `tsc`
  clean both repos, `next lint` clean on the changed frontend file.
- 2026-09-08: User asked to drop "Actual" from "Actual Supplier Window" — trivial rename, applied
  directly (header string only).
- 2026-09-08: User asked whether renaming "Days" to "Days since arrived" would be fine, or if there
  was a better option, flagging it as confusing next to "Return Days Left" — an opinion request,
  not a go-ahead ([[feedback_opinion_request_is_not_a_go_ahead]]). Recommended "Days Since Arrival"
  (Title Case consistency, and confirmed the column's meaning — days since `partArrivedDate` — is
  the same across all 8 buckets sharing this panel, not just the two touched by this ticket) and
  stopped for confirmation. User said "yes go ahead" — applied, lint-checked.
- 2026-09-08: User asked for commit messages for both repos, then said "mark the ticket AG-284 as
  done" — closed, no further changes.
