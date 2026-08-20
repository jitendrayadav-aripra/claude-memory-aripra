---
name: issue-AG-245-legacy-stock-supplier-parts-merge
---

## NOW

- **Status (2026-08-20): DONE (closed by user).** Ticket closed at the user's explicit instruction.
  Factual note for any future session, kept here so this doesn't get misread as "feature live":
  the code itself is IMPLEMENTED-THEN-REVERTED, not currently in the codebase (same pattern as
  [[issue-AG-243-consumables-hover-info-tooltips]] the day before) — confirmed via grep across all
  3 touched files, none of the merge logic/type changes/badge remain.
- **Requirement:** following AG-198 (hid the "STOCK" pseudo-supplier from the RFQ picker), show
  historical parts whose supplier was "STOCK" — previously visible only in the Parts Leaderboard —
  ALSO in the Parts Tab's "Stock" list, folded into its Total Cost / Net Saved Money KPIs. Explicit
  constraint: do not change the Parts Leaderboard at all.
- **Key architectural finding (durable, worth keeping even though the code was reverted)**: the
  *new* non-consumables "fulfil from stock" flow (`fulfilTaskPartFromConsumableStock`,
  `car-planet-backend/server/services/inventory/inventory.service.ts:2908-3055`) already writes
  into the same `consumable_transaction` table (via the AG-236 consumable-deduction chain) that
  powers the Parts Tab's "Stock" list (`consumableStockPartsList`/`Summary`, same file
  `:3066-3237`) — so every NEW stock-fulfilled part, consumable or not, already lands there
  correctly today. The only real gap was historical STOCK-supplier rows, which live entirely in
  `task_part`/`po`/`supplier` (a structurally different table, no shared key). The Parts Leaderboard
  query (`buildPartsLeaderboardQuery`, same file `:3980-4125`) already includes STOCK-supplier rows
  unfiltered — confirmed needs zero changes to satisfy the "don't touch the dashboard" constraint.
- **Exact field mappings confirmed in the `TaskPart` entity** (useful if this is redone):
  requested date → `taskPart.partRequestedDate`/`partRequestedDateText` (date filter for merged
  rows runs on this, per user's explicit instruction — not an arrival/transaction date); issued/
  arrived date → `taskPart.partArrivedDate`/`partArrivedDateText`; requested by →
  `taskPart.createdBy` (User relation); cost → `taskPart.poUnitPrice`; dealer price →
  `taskPart.dealerPrice`.
- **Approach used (ready to reapply if asked):** new `buildLegacyStockSupplierTaskPartsQuery` +
  `mapLegacyStockSupplierTaskPart` in `inventory.service.ts`, joining `TaskPart → po → supplier`
  filtered on `supplier.company = 'STOCK'` (mirrors `buildPartsLeaderboardQuery`'s own join
  pattern), mapped into the exact `ConsumableStockPartItem` shape the Stock list already uses, with
  a `isLegacyStockSupplier: true` flag. `consumableStockPartsSummary` folds these rows' cost/saved
  into totalCost/totalSaved ONLY — user explicitly said no "Total Parts" count KPI needs to grow.
  `consumableStockPartsList` had to move from SQL-level pagination/sorting to JS-level (merge two
  structurally different queries, sort in JS, then slice) since there's no common ORDER BY across
  `consumable_transaction` and `task_part`. IDs from both sources were prefixed (`ct-`/`tp-`) since
  they come from different tables/sequences and could numerically collide once merged — this
  required changing `ConsumableStockPartItem.id` from `number` to `string` in `parts.api.ts`.
  Frontend `consumable_stock_parts_list.tsx` renders a "Supplier: Stock" badge next to the part
  name when `isLegacyStockSupplier` is true.
- **3 files touched:** `car-planet-backend/server/services/inventory/inventory.service.ts`,
  `carplanet/src/app/dashboard/leaderboard/parts-tab/parts.api.ts`,
  `carplanet/src/app/dashboard/leaderboard/parts-tab/consumable_stock_parts_list.tsx`.
- **Don'ts:** don't assume any of this merge logic is live right now — verify current file state
  (`grep isLegacyStockSupplier` in the 3 files above) before building on top of this or telling the
  user it's live.

---

## HISTORY

- 2026-08-20 — User gave the sole context as `parts-tab.tsx` + a plain-English description
  referencing AG-198 (hid STOCK supplier). Ran a thorough Explore agent to map the actual
  architecture before proposing anything, since it wasn't obvious whether the Parts Tab's "Stock"
  list (labelled just "Stock" in the UI, named `showConsumableStock`/`ConsumableStockPartsList` in
  code) was consumables-only or a general stock list — confirmed it's structurally consumables-
  shaped but functionally already the unified destination for all new stock-fulfilled parts
  (consumable or not), leaving only historical STOCK-supplier task_part/po rows as a genuine gap.
  Presented the finding + 3 clarifying questions (field-mapping gaps for requested/issued
  date/requested-by, visual distinction for merged rows, KPI scope) in prose; user answered
  precisely, confirming real entity fields (`partRequestedDate`/`partArrivedDate`) existed for the
  date concepts. Implemented the 3-file merge, `tsc --noEmit` clean on both repos (0 errors) both
  times checked. By the time the user asked to mark this done, all 3 files had reverted back to
  pre-session state in the working tree (confirmed via grep) — same pattern as AG-243 the day
  before. Per harness instruction, not reverting back or raising it with the user; filed as a
  designed-and-verified-but-not-currently-live reference, closed at the user's explicit request.
