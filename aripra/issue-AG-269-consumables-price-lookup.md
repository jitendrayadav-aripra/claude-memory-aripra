---
name: issue-AG-269-consumables-price-lookup
---

## NOW

- **Status: DONE (closed) 2026-09-04.** `tsc` clean (backend). Closed by explicit user instruction.
- **Scope changed from the original ticket text before building** — the user gave a NEW requirement
  on resume (2026-09-04), overriding the ticket's own "merge Adjustment-In into the existing
  most-recent-wins resolution" acceptance criteria: instead, a **strict per-product priority**,
  Invoice price > PO unit price > manual stock-adjustment price > no price — each tier resolved
  independently ("most recent within the tier"), combined by priority, NOT by recency across mixed
  sources. This is a real behaviour difference from what the ticket originally asked for: under the
  old plan, a newer PO-only procurement row could still out-rank an older row's real invoice price;
  under what was actually built, invoice price always wins for that product regardless of recency.
  If this ticket is ever compared back against its own Jira acceptance criteria, flag that the built
  behaviour is the user's later verbal instruction, not the original AC text.
- **Implemented in `resolveLastPriceByProductId()`** (`inventory.service.ts:5714`, corrected line —
  the ticket's own text had a stale `:5560` reference): 3 independent tiers
  (`invoicePriceByProductId`/`poPriceByProductId` from `ConsumableProcurement`,
  `adjustmentPriceByProductId` from `ConsumableTransaction` `Adjustment In` rows, newly added), then
  `invoice ?? po ?? adjustment` per product. `ConsumableTransactionType` import added to
  `inventory.service.ts`.
- **No backfill/migration** — confirmed via direct investigation: `ConsumableProduct` has zero price
  columns, nothing anywhere caches a resolved price, `resolveLastPriceByProductId()` recomputes fresh
  from live data on every call (all 3 callers: `getConsumablesOnHandStats`,
  `getConsumablesCategorySummary`, `getConsumablesStockList`). The new logic applies to all 726
  products' historical data immediately, no script needed. User asked this exact question before
  giving permission to build — answered and confirmed correct via code investigation, not assumption.
- Documented in `my-docs/projects/parts-and-consumable-inventory-4th-project/NEW_PARTS_AND_STOCK_INVENTORY.md`
  (new "Related ticket — AG-269" section + changelog entry, 2026-09-04), per explicit user instruction
  to keep that doc current on "how the prices are being shown."
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
