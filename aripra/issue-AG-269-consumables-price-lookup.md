---
name: issue-AG-269-consumables-price-lookup
---

## NOW

- **Status (2026-09-03): DEFERRED — analysed and verified, not started.** User said "we will do it
  later." No branch/code changes yet. Next action when resumed: implement the fix below, plan in
  `todo.md` first per project workflow rules, get sign-off, then build.
- **Ticket:** AG-269, `Consumables tab price lookup ignores manually-entered stock-adjustment
  prices` (Story, child of Epic AG-255 "Parts Inventory — Leakage & Accountability Dashboard").
  Reporter/creator: Akash Robert. Status: Backlog. Priority: High ("fix this first" — must land
  before a separate price-backfill ticket whose recovery-% math depends on today's no-price count).
- **Bug:** Parts Oversight → Consumables tab shows 99 of 726 products as "no price" even though
  those 99 already have a real, human-entered price on a stock-adjustment transaction.
- **Root cause (verified in code, not just trusted from the ticket text):**
  - `resolveLastPriceByProductId` (`car-planet-backend/server/services/inventory/inventory.service.ts:5560`)
    powers the Consumables tab (category tiles + stock-list) via
    `POST /inventory/parts-inventory-consumables-category-summary` and
    `.../parts-inventory-consumables-stock-list` (`inventory.router.ts:142-147`), called only from
    `carplanet/src/app/dashboard/leaderboard/parts-inventory/consumables/consumables_tab.tsx`
    (confirmed — grepped the whole frontend, no other caller).
  - That function reads **only** `ConsumableProcurementRepository`. It never reads
    `ConsumableTransaction.unitPrice`, the field `addStockAdjustment`
    (`consumable.service.ts:577-653`) writes for manual stock-in adjustments.
  - A correct merge pattern already exists in `getConsumableRfqDetail`
    (`consumable.service.ts:2334-2452`), which unions Procurement Restock rows with Manual
    Adjustment-In `ConsumableTransaction` rows for a different screen's "Best Price Paid". Note:
    that function's own tie-break picks the **lowest** price ever paid — a different rule for a
    different purpose. The ticket correctly scopes reuse to just the *source-merging pattern*
    (Procurement + Manual Adjustment-In), not that tie-break.
  - `resolveLastPriceByProductId`'s own existing tie-break is "most recent wins" (orders
    procurement rows ascending by date, overwrites a Map on each iteration so the last/most-recent
    write survives) — the acceptance criteria says the new Adjustment-In prices should merge into
    *that* same most-recent-wins resolution, not adopt the RFQ function's lowest-price rule.
- **Fix (acceptance criteria):** extend `resolveLastPriceByProductId` to also resolve price from
  `ConsumableTransaction` rows where `transactionType = 'Adjustment In'` and `unitPrice IS NOT
  NULL`, merged into the same most-recent-wins resolution already used for procurement prices.
- **Verified true:** every code-level claim in the ticket (function names, line ranges, caller
  graph, what each function reads/writes) checked out exactly against the current repo. Could NOT
  verify the raw data claim (99/726, 89 with one consistent price + 10 with minor revisions) — the
  MySQL MCP connector for this DB isn't authorized in-session; that part rests on the ticket
  author's direct DB check, not independently confirmed.
- **Don't:** don't port `getConsumableRfqDetail`'s lowest-price tie-break — only its idea of which
  two sources to merge.

---

## HISTORY

- 2026-09-03 — Fetched AG-269 from Jira (`aripra.atlassian.net`, cloudId
  `761f8a59-91ff-493b-ab27-a38827186c27`) on request to analyse it. Verified every code claim by
  reading `inventory.service.ts`, `consumable.service.ts`, `inventory.router.ts`, and grepping the
  frontend `parts-inventory` folder for callers. Confirmed accurate. User deferred implementation
  ("we will do it later") — saved for resume via `issue-AG-269-*`.
