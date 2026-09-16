---
name: stock-price-estimation-analysis-docs
description: Where the AI-estimated stock/consumable "actual price paid" research analysis lives, and its key findings — for AG-271 (sub-task of AG-269, "can AI estimate the missing price of stock already on hand?")
metadata:
  type: reference
---

**Where the docs live:** `my-docs/projects/parts-and-consumable-inventory-4th-project/`:
- `STOCK_ACTUAL_PRICE_PAID_ESTIMATION_ANALYSIS.md` — the main analysis. NOT resale-price estimation
  (that's the separate `AI_RESALE_PRICE_ESTIMATION.md` in the same folder, a different topic/ticket)
  — this is about estimating the *acquisition/cost* price for `consumable_product` rows that have
  never gone through a priced purchase event at all.
- `stock_as_supplier_price_analysis.md` — a follow-up checking whether prices already recorded on
  `task_part` rows with `supplier = 'STOCK'` (the separate vehicle-job Parts/PO flow) can be borrowed
  for matching consumable products, instead of/ahead of AI estimation.

**Key findings, so a future session doesn't have to re-derive them:**
- The price gap is real and structural: `consumable_product` has no price column of its own — price
  only exists via `resolveLastPriceByProductId()`'s 3 tiers (invoice price / PO unit price / manual
  "Adjustment In" stock-adjustment price). No purchase history = no price, full stop.
- **Production, verified directly (not local — local dev data is far too sparse to be
  representative):** 505 active products, 195 already priced, **310 with no price at all**.
- Of those 310: 47.7% have a price-like figure written in `description`, 98.1% have *some* `sku`
  value, 11% have a composite/cross-reference-looking `sku`, 7.7% have a recognizable brand in the
  name, and only ~1.6% have no signal anywhere.
- **No new system needed** — Google Gemini is already integrated and doing near-identical work today
  (`ai-dealer-price.service.ts`'s `suggestDealerPrice`, plus `consumable_transaction.dealer_price`,
  plus `advert-gemini-spec.service.ts`'s structured-JSON-extraction pattern). This would be new
  prompts/call sites on existing infrastructure, not a new integration.
- The `supplier = 'STOCK'` Parts/PO idea works but is narrow: only ~8% of the 310 (25 products) get
  an exact name match, and reliability varies hugely by product — generic names like "oil filter"
  (117 distinct prices, £0.01–£15.99 range) are too ambiguous to trust without a real part number;
  tighter matches like "front wiper blade stock" (£5.68–£6.20) are usable. `£0.01` recurring as the
  minimum across many high-volume names is almost certainly a data-entry placeholder, not a real
  price — must be filtered before averaging.
- Recommended layering: (1) the STOCK-supplier name-match as a free first pass where tight enough,
  (2) AI description-mining (Tier A), (3) AI SKU/part-number cross-reference (Tier B), (4) AI
  name/brand/category estimate (Tier C, weakest, last resort) — never auto-apply low-confidence
  results, same "labelled as an estimate" convention already established for the resale-recovery
  feature (AG-293/296).

**Not yet a ticket** — this was ad-hoc research requested directly in chat, not filed under AG-271 in
Jira yet. If resumed, check whether AG-271's Jira description/status has moved before assuming this
is still "research only."
