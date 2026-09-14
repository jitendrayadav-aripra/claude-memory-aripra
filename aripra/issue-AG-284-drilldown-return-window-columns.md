---
name: issue-AG-284-drilldown-return-window-columns
description: AG-284 — First Flagged/Arrival Date, Supplier Window, and Return Days Left columns on the Awaiting Supplier Return and Return Window Unknown Overview drilldowns
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-11.** Previously closed 2026-09-08, resumed+re-closed 2026-09-09,
bounced back again 2026-09-11 for the 3-bucket extension below — built, refined, and re-closed same
day. `tsc`+`next lint` clean both repos. Part of the Parts Oversight module — documented in
`NEW_PARTS_AND_STOCK_INVENTORY.md` under "Related ticket — AG-284".

**2026-09-11 — new requirement, full Overview-tab column audit found this ticket only covers 2 of
the 4 resolution-stage buckets sharing this same drilldown widget/mechanism
(`parts_inventory_drilldown_panel.tsx`, `extraColumns` keyed on `params.resolutionStage`). Extend to
the other two:**
- **Awaiting eBay Listing (`READY_FOR_SALE`)** — add First Flagged Date + days past return window.
  Per the user, backend already selects `firstFlaggedDateText`/`returnWindowDays` for every bucket
  (`inventory.service.ts:6410-6413`) — claimed frontend-only, no backend change. **Needs
  verification against current code before trusting** — line numbers/claims from tickets have
  drifted before this session (AG-287/AG-288 both touched this same file).
- **Listed on eBay – Unsold (`ALREADY_ON_SALE`)** — add Listed Date + Listed By. Per the user, both
  fields exist on `TaskPart` and are already batch-fetched for AG-260/261/263
  (`inventory.service.ts:797-816, 1005-1034`) but NOT yet selected into
  `getPartsInventoryDrilldown` for this bucket — needs a new select + a new `extraColumns` branch.
- **Return Window Unknown** — diagnostic distinction wanted between "no PO linked" vs. "supplier has
  no return window set" (currently one undifferentiated bucket). `po`/`poSupplier` already joined
  per the user (`inventory.service.ts:6247-6248`) — needs verification + a derived
  flag/label, not just a raw column.
- **Also requested: sorting on the (new and existing) columns** in this panel — likely reuses the
  `sorting`/`sortMap` mechanism just added backend-wide in [[issue-AG-288-column-sorting-gaps]].

**2026-09-11 — built, `tsc` clean both repos + `next lint` clean.** No migration (all fields already
existed on `TaskPart`; only new selects/joins). Confirmed the ticket's own line-number references had
drifted (e.g. quoted `6247-6248` for the po/poSupplier join actually landed in an unrelated People
function; the real join is inside `getPartsInventoryDrilldown` itself) — re-verified against live
code before building, not trusted as given.
- **`inventory.service.ts` (`getPartsInventoryDrilldown`)**: added `.leftJoin(User, "listedByUser",
  "listedByUser.id = taskPart.listedBy")` (listedBy is a plain int, no relation — same precedent as
  `vehicle.service.ts:16210`'s `mechUser` join, chosen over a post-query batch-fetch specifically so
  "Listed By" stays sortable in SQL before the 200-row cap). New selects: `po.id` (as `poId`),
  `taskPart.listedDateText`, `listedByUser.firstName/lastName`. `sortMap` gained 4 entries:
  `daysPastWindow` (mirrors `returnDaysLeft`'s expression, negated), `listedDate`, `listedBy`
  (CONCAT, same convention as `technician`/`requestedBy`), `reason`.
- **`parts_inventory_drilldown_panel.tsx`**: 3 new `extraColumns` branches —
  - `READY_FOR_SALE` ("Awaiting eBay Listing"): First Flagged Date (reused `firstFlaggedDate`
    sortKey) + "Days Past Return Window" (`days - returnWindowDays`, always defined here since this
    bucket's own query filter guarantees both non-null — genuinely frontend-only, matching the
    user's own note).
  - `ALREADY_ON_SALE` ("Listed on eBay – Unsold"): Listed Date + Listed By.
  - `UNKNOWN` ("Return Window Unknown"): added a 3rd column, "Reason".

**2026-09-11 follow-up (same day) — multi-reason correction.** User caught that the initial
single-priority `getReturnWindowUnknownReason()` (pick one of 3 mutually-exclusive-seeming causes)
was wrong: "no arrival date" and "PO/window missing" are actually **independent** — a row can be
missing both at once, and picking just one would silently hide the other. Rebuilt as
`getReturnWindowUnknownReasons()` returning a string array (pushes "No arrival date recorded" if
`partArrivedDate` is null, and separately pushes "No PO linked"/"Supplier has no return window set"
if `returnWindowDays` is null), joined with `" + "`. Per the user's own explicit ask, kept the cell
compact when both apply: `className="block max-w-[220px] truncate"` + a native `title` attribute
carrying the full joined text for hover, rather than letting the row wrap/widen. Backend's `reason`
sortMap rank changed from a 3-way priority CASE to a combination rank (`arrival-missing? +4` plus
`no-PO? +2 : no-window? +1 : 0`) so rows sharing the same combination of reasons sort together —
single-priority ranking would have scattered "both missing" rows arbitrarily between the two
single-cause groups. `tsc`+`next lint` clean both repos after this correction too.

Not yet manually verified in the running app by the user beyond this review — closed by explicit
"mark the ticket AG-284 as done" instruction.

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
- **"Awaiting Supplier Return" (`WAITING_TO_BE_RETURNED`)** — 4 columns, in order: Parts Arrived
  Date, First Flagged Date, Supplier Window, Return Days Left. (Order swapped 2026-09-09 — see
  below; originally shipped First Flagged Date first.)
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
  Date. Both drilldowns now lead with the arrival date then the flagged date (see 2026-09-09
  follow-up below) — a deliberate real-world sequence (arrive, then later get flagged), not a
  coincidence of column order.
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
- 2026-09-09: User resumed the ticket for one small change — swap "Awaiting Supplier Return"'s
  column order so Parts Arrived Date leads First Flagged Date (reflecting the real sequence: a
  part arrives, then is later flagged faulty/incorrect). Applied directly (array-order swap +
  updated the file's own explanatory comment, which had described the old First-Flagged-first
  order as deliberate). `tsc`+`next lint` clean. User then said "mark the ticket as done" — closed
  again, no further changes.
