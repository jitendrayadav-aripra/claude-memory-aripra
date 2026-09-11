---
name: issue-AG-287-overdue-fitting-instock-statuses
description: AG-287 — "Overdue fitting" 2-day-breach metric only counts vehicle.status = "In Stock", missing 6 other physically-in-stock statuses
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-11.** `tsc` clean (backend-only). No migration. Closed by explicit
instruction. Already fetched via Jira in the prior session turn
(while updating the Release 2 doc, which now references this ticket as an in-progress item under
"Overdue-Fitting Check Doesn't Yet Cover All In-Stock Vehicle Types"). Sibling to
[[issue-AG-286-parts-arrived-sold-cars]] (same Overview KPI family, same reporter).

**Core ask:** the "overdue fitting" / breaching-2-day-rule metric currently only flags parts on
vehicles with `status = "In Stock"` exactly. Akash confirmed 6 more statuses are also physically
in-stock and should count: Deposit, Reservation, Trade Stock, Courtesy Car, Company Car, Awaiting
Payment. Fix needed in all 4 places this predicate lives (per the ticket's own dev notes — needs
re-verification against current code, line numbers have drifted since AG-285/286/289 all touched
`inventory.service.ts` this session).

**Ticket explicitly flags 5 further open business questions, NOT part of the Acceptance
Criteria, marked "do not build against a guess":** Cancelled, Walkaway/Awaiting/Rescheduled/Buyers
Visit Booked, SOR (Sale or Return), Aftersales, Campaign. These are genuinely unresolved — need to
surface to the user as real doubts, not silently include or exclude.

**Confirmed all 4 sites against current code** (line numbers drifted from AG-285/286/289):
- `getPartsInventoryPartsStats` (~5521/5553-5557/5606) — KPI count/value.
- `getPartsInventoryDrilldown`'s `BREACHING_TWO_DAY` case (~6350).
- `getPartsInventoryBreachList` (~6695) — the Alert Checks breach table.
- `getPartsInventoryReadyCars` (~6814) — the "nothing fitted" vehicle-grouped table. Initially
  wondered if this 4th site was really in scope (it's conceptually a different Alert Checks table),
  but the ticket's own AC explicitly says "the two table query sites" — confirmed both belong.

**Confirmed the exact status values to reuse** (not hand-typing new literals):
`OCCUPANCY_COUNTED_STATUSES` (`vehicle-movement-planner.const.ts:77`) = `[IN_STOCK, DEPOSIT,
RESERVATION, TRADE_STOCK, COURTESY_CAR, COMAPNY_CAR]` (note: `COMAPNY_CAR` is a real typo already
in the live enum — preserved as-is, not "fixed"). `AWAITING_PAYMENT` lives in the same `StatusEnum`
(`vehicle.interface.ts:126`) but isn't part of that constant — combine both into one new shared
constant, matching the `EXCLUDE_POST_HANDOVER_AFTERSALES`-style single-constant pattern already
established this session (AG-286) to avoid the exact drift risk AG-286/287 both call out.

**The 5 open business questions are real doubts, not yet answered — surfaced, not guessed:**
Cancelled, Walkaway/Awaiting/Rescheduled/Buyers Visit Booked, SOR, Aftersales, Campaign. None of
these are in the AC's confirmed 7-status list. User confirmed: build only the AC-confirmed 7
statuses; the 5 stay an open follow-up, not built.

**Built:** new shared constant `OVERDUE_FITTING_IN_STOCK_STATUSES` (`inventory.service.ts:145`) =
`[...OCCUPANCY_COUNTED_STATUSES, StatusEnum.AWAITING_PAYMENT]`, imported from
`vehicle-movement-planner.const.ts` / `vehicle.interface.ts`. Applied at all 4 confirmed sites via
`vehicle.status IN (:...inStockStatuses)`, replacing the old `= :inStockStatus` single-value bind.
Removed the now-dead local `IN_STOCK_VEHICLE_STATUS` constant in `getPartsInventoryPartsStats`.

**Verified against the live local database** — no MCP MySQL tool was actually configured on this
machine despite the backend `CLAUDE.md` describing one (`.mcp.json` is gitignored and simply didn't
exist here); connected directly instead via a scratchpad Node script using `mysql2` (already a
transitive dependency) and `car-planet-backend/.env` credentials, the same approach used in an
earlier session before any DB MCP tool existed at all. First attempt hit a real bug: `.env` has
CRLF line endings, and `line.match(/^(...)=(.*)$/)` silently matched **zero** lines — JS `.` never
matches `\r`, so the trailing `\r` from `split("\n")` blocked `$` from ever reaching true end-of-
string. Fixed by splitting on `/\r?\n/` instead. Result: **63 parts, £3,033.05** newly qualify
across the 6 added statuses (Deposit 27 · Trade Stock 14 · Company Car 13 · Courtesy Car 5 ·
Awaiting Payment 3 · Reservation 1) that weren't visible on this metric before the fix.

**Related:** [[issue-AG-286-parts-arrived-sold-cars]] (sibling ticket, same reporter, same Overview
KPI family, built the same session).

---

## HISTORY

- 2026-09-11: Ticket opened by user request ("New Ticket: AG-287"), memory file created immediately
  per [[create-ticket-file-immediately-on-open]] before any analysis.
- 2026-09-11: Traced all 4 predicate sites and the exact status constants to reuse. Explained
  understanding, surfaced the 5 open business questions as real doubts (per the ticket's own "do
  not build against a guess" instruction) rather than picking a default, and asked permission. User
  confirmed: build only the 7 AC-confirmed statuses. Built the shared constant + all 4 sites. `tsc`
  clean. Gave a short commit message proactively.
- 2026-09-11: User asked for a SQL Workbench query to verify, then asked for the data directly
  rather than a query to run themselves. Found the tool available was `mysql_prod` (production), not
  local — flagged the mismatch rather than silently querying production. User asked why no local DB
  access existed given a prior session apparently had it — investigated and found `.mcp.json` is
  gitignored and doesn't exist on this machine; the "prior access" was actually a direct `mysql2`
  script against `.env` credentials, no MCP tool involved at all (matches AG-260's own memory
  history: "direct read-only mysql2 query... no MCP DB tool available"). Built the same way here.
  First run failed with "Access denied for user ''@'localhost'" — root-caused to a CRLF parsing bug
  in the `.env` reader (see NOW), fixed, reran successfully. Auto-mode classifier blocked the first
  execution attempt (direct DB connection flagged as sensitive) — explained what/why to the user,
  got explicit "yes go ahead" before it actually ran. Reported the 25-row sample, then the user
  asked for the real total — ran a COUNT/SUM+GROUP BY follow-up query, reported 63 parts/£3,033.05.
- 2026-09-11: User asked to update `NEW_PARTS_AND_STOCK_INVENTORY.md` — added the related-ticket
  section + changelog entry including the verified figures. User then said to mark AG-287 done and
  move to a new ticket.
