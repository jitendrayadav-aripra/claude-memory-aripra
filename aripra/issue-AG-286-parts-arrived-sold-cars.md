---
name: issue-AG-286-parts-arrived-sold-cars
description: AG-286 — exclude parts on post-handover Aftersales Request tasks from the "Parts arrived on sold cars" leakage metric
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-10.** `tsc` clean backend (frontend untouched — pure backend fix,
confirmed pass-through display). Closed by explicit instruction. No migration.

Bug: the "gone car" bucket (Overview KPI tile **"Parts arrived on sold cars"**, its drilldown, and
the Alert Checks "Car gone" list) flagged ANY arrived-unused part on a Sold/Refunded/Cancelled
vehicle — including parts on a genuine after-sales repair job (customer already owns the car, has
an issue, a part is being bought to fix it). That's not stranded/leakage stock.

**User's question, answered by tracing code rather than guessing:** asked whether "booking date"
plays a role (later corrected to "handover date"). Booking date plays no role anywhere. Found a
related-but-distinct precedent while checking — `vehicle.service.ts`'s
`AFTERSALES_PARTS_GREEN/RED/GRAY` model (~5789-5830) compares `task.taskDateTime >
MAX(invoice.handover_date)` per vehicle to decide if a task counts as genuine post-sale work. The
ticket's own Acceptance Criteria only asked for a category-only exclude (no date gating) — flagged
this explicitly as an open decision rather than silently picking one. **User chose category +
handover-date-gated**, matching the stricter existing precedent.

**Built:** one shared constant, `EXCLUDE_POST_HANDOVER_AFTERSALES` (`inventory.service.ts:120`,
right after `TERMINAL_RESOLUTION_STATUSES`) — excludes a part only when its task's category is
`'Aftersales Request'` **AND** `task.taskDateTime > MAX(handover_date)` for that vehicle (reuses
the exact same `MAX(handover_date)`-per-vehicle SQL pattern as the `AFTERSALES_PARTS_GREEN` model,
for consistency). A part on a pre-handover Aftersales Request task, or with no handover date
recorded at all, still counts as leakage — conservative default, can't prove it's legitimate
aftersales work without a real handover date to compare against.

Applied identically to **all 3 places** (avoids the drift risk the ticket itself flagged — this
predicate was duplicated 3x with no shared helper before):
- `getPartsInventoryPartsStats` (~5445) — the KPI tile's SUM. Had no `task`/`taskCategory` join at
  all before this — both added.
- `getPartsInventoryDrilldown`'s `ON_GONE_CARS` case (~6306) — `task` was already joined (for other
  buckets), only `taskCategory` needed adding.
- `getPartsInventoryGoneList` (~6662) — **correction to the ticket's own claim** ("no task join at
  all") — `task` was already joined (for the row-click "open Task Card" action), only
  `taskCategory` needed adding.

**Related:** none directly — standalone accuracy fix on the resolution/leakage metrics built under
AUT-3572, touches the same Overview tile documented there.

---

## HISTORY

- 2026-09-10: Ticket opened by user request ("new ticket: AG-286"), memory file created immediately
  per [[create-ticket-file-immediately-on-open]] before any analysis — first ticket this session
  where that rule was actually followed from the start.
- 2026-09-10: Fetched from Jira, traced all 3 affected query sites against current code (line
  numbers had drifted from AG-284/285/261 work since the ticket's own notes were written — found
  and corrected one stale claim in the ticket itself about `getPartsInventoryGoneList`'s joins).
  User asked specifically whether "booking date" plays a role — traced into
  `vehicle.service.ts`'s `AFTERSALES_PARTS_GREEN` model to check properly rather than answer from
  assumption, found it uses `task.taskDateTime` vs `invoice.handover_date` for a related-but-
  different purpose. Explained understanding + plan, flagged the date-gating question as a genuine
  open decision (category-only vs category+date-gated) rather than assuming either way.
- 2026-09-10: User corrected "booking date" → "handover date" (typo, same question). Confirmed the
  two-option framing still applied with corrected terminology.
- 2026-09-10: User chose category + handover-date-gated (the stricter option, matching the existing
  precedent). Built one shared SQL constant reused across all 3 query sites instead of copy-pasting
  the condition three times, per the ticket's own flagged drift risk. `tsc` clean. Gave short commit
  messages proactively (no reminder needed this time).
- 2026-09-10: User said to mark AG-286 done and update `NEW_PARTS_AND_STOCK_INVENTORY.md`.
