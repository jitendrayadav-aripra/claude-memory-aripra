---
name: issue-AG-271-ai-price-estimation-research
description: AG-271 — research/build whether AI or heuristics can estimate the missing "actual price paid" for consumable_product rows with no purchase history; sub-task of AG-269
metadata:
  type: project
---

## NOW

**Status: in progress, resumed 2026-09-21.** Description-mining pass is DONE; user's next instruction
is to move on to "other methods" — not yet specified which one. Retroactively creating this ticket
file now since real work has accumulated across multiple sessions without one (previously tracked
only as [[stock-price-estimation-analysis-docs]] reference memory).

**Live Jira description (verified 2026-09-16, differs from my own earlier chat-only research —
see below):** "Research whether AI can estimate the price of stock already on hand. This story fixes
one confirmed gap (manual stock-adjustment prices never read), recovering 99 of 726 no-price
consumable lines. Two heuristic approaches were evaluated separately for the rest: sibling-name
matching (about 1.7% recovery) and free-text notes mining (up to about 59% with human review). See
the stakeholder deck and supporting workbook for that analysis. Before building either, check
whether an AI/LLM approach would do better... This is research only, not a build commitment."
Status: In Progress. **I do not have the stakeholder deck/workbook** referenced — could not
reconcile its "726 lines" / "99 recovered" / "1.7%" / "59%" figures against the live schema (see
HISTORY). Treating my own live-production numbers as the working baseline instead, per user's
2026-09-18 instruction to "just do a live analysis of the production database."

**Live baseline (verified against production, 2026-09-18):** `consumable_product` — 509 Active, 288
Deleted. Of the 509 active: 202 priced (via `resolveLastPriceByProductId()`'s 3 tiers — invoice /
PO unit price / manual "Adjustment In" adjustment), **307 with no price**. A legacy `inventory_stock
WHERE type='C'` table (697 rows, no price column, explicitly disconnected from the current system
per its own entity comment) is the likely source of the Jira ticket's "726" figure — not reconciled.

**Method 1 (description-mining) — DONE 2026-09-18/19.** Queried all 307 no-price rows' `description`
field live from production, wrote a Node script (`mine_prices_v2.js` in
`C:\Users\jiten\AppData\Local\Temp\claude\docxgen\`) to extract price candidates:
- Keyword-associated numbers ("unit price", "price per unit", "new price", "latest price") — High
  confidence.
- Bare currency-symbol amounts with no keyword, or a bare leading number with no unit-of-measure
  word after it — Medium confidence.
- **146 of 307 (48%) had at least one mineable price** — close to the ~47.7% signal estimated in
  the original chat-only research.
- User caught a real gap in the first pass: when a description listed more than one price (e.g.
  multi-supplier quotes), the script kept only the last/chosen one — fixed to list ALL distinct
  values found, comma-separated, per row.
- Deliverable: `my-docs/projects/parts-and-consumable-inventory-4th-project/Consumable_No_Price_Mined_From_Description.docx`
  — S.No, Product, SKU, Mined Price(s), Confidence, Source text columns, verified by extracting the
  docx and checking multi-price rows render correctly.
- 161 of 307 had no mineable price at all (empty description, pure quantity text like "UNITS"/"2litre",
  or a bare part-number-looking token) — correctly excluded, not fabricated.

**Method 2 (SKU/OEM cross-reference) — DONE 2026-09-21.** Matched each no-price product's `sku`/
`oem_number` (exact, normalized) against (a) other `consumable_product` rows with a resolvable price,
and (b) `task_part` rows (invoice price falling back to PO price). **41 of 307 matched**, but **30 of
those 41 were already covered by description-mining** (same product id, not a re-match by name/SKU —
checked overlap by product id specifically, per user's explicit ask to confirm this). Only **11 were
genuinely new coverage**, and of those 11, **7 were Low confidence** (generic placeholder SKUs like
"paper"/"mop"/"foam" shared across genuinely different variants — verified case: "paper" matched BOTH
an 18" and 36" masking-paper product for what's actually a 48" product). Deliverables (3 docs, since
user wanted the full-41 and new-11-only kept separately): `Consumable_No_Price_Matched_Via_SKU_Crossref.docx`,
`..._NewCoverageOnly.docx`.
**Running combined distinct total after methods 1+2:** description-mining's true count is 147 (146
scripted + the 1 manual id-18588 addition) + 11 new-from-SKU = **158 of 307 (51%) priced**, leaving
**149 remaining** — this 149 (not the 146-only figure quoted earlier in chat) is the exact pool
methods 3 and 4 below actually ran against.

**Method 3 (STOCK-supplier `task_part` name-match) — DONE 2026-09-21, null result.** For the
remaining pool (149 products after excluding all description-mined + new-SKU-matched ids), matched
by exact normalized product NAME against `task_part` rows with `supplier = 'STOCK'` and a resolvable
price. **Only 2 of 149 had any name match at all** (both generically named "Oil filter"), and the only
price signal available (£0.01–£15.99 across ~1,980 rows) was dominated by the same `£0.01` placeholder
pattern already flagged in the original chat research — confirmed, not just suspected, via the actual
price-frequency distribution. **0 of 149 got a trustworthy price.** Deliverable:
`Price_Estimation_STOCK_Supplier_Name_Match.docx` — short, client-facing, documents the null result
with evidence rather than silently skipping it.

**Method 4 (AI name/brand/category estimate via Gemini) — DONE 2026-09-21.** Built a one-off local
Node script (NOT part of the app — reads `car-planet-backend/.env`'s real `GOOGLE_API_KEY`/
`GOOGLE_GEMINI_MODAL`, calls Gemini directly, no DB access either direction), modeled on
`ai-dealer-price.service.ts`'s input shape + `advert-gemini-spec.service.ts`'s output shape
(`responseMimeType: "application/json"` + `responseSchema`, `temperature: 0`, retry w/ backoff).
`sku` passed to the prompt as "Part Number" per user's explicit instruction.
**Calibration first** (8 products with a known real price, description stripped so it's a blind
test): Gemini overestimated 6 of 8 by 24%–131% (average ~50-60% too high), and its own stated
confidence didn't correlate with actual accuracy (both near-exact hits were "Medium," both "High"
answers were still 24-42% off). Reported this honestly before scaling up.
**User's explicit instruction after seeing the calibration**: run it anyway on all 149, but leave the
Confidence column blank in the deliverable — the point for the client is that even AI can't pin these
down, evidencing a data-entry/record-keeping gap, not delivering usable prices. Ran all 149 (one
bookkeeping slip caught and fixed: the input pool file accidentally had 162 rows — 13 already-priced
products included by mistake — filtered back down to the correct 149 at doc-build time, no data lost).
Deliverable: `Consumable_No_Price_AI_Estimated.docx` — price + Gemini's own reasoning per product,
Confidence column present but intentionally empty, framed for the client as evidence of the data gap.

**Final tally across all 4 methods:** 147 (description, incl. 1 manual) + 11 (new SKU) + 0 (STOCK
name-match) = **158 of 307 have a real, defensible price**. The remaining **149** have only an AI
*estimate* (Method 4), explicitly not presented as reliable — Confidence intentionally left blank in
that deliverable.

**Status: DONE (closed) 2026-09-21 in memory.** All 4 methods complete, all deliverables produced and
verified. Jira ticket status NOT touched (memory-only close, per
[[mark-as-done-means-memory-not-jira]]) — AG-271's live Jira status remains "In Progress" with its
original description; not updated to reflect this session's findings unless separately asked. The
master-merge of all per-method tables into one combined list was offered but not yet requested.

---

## HISTORY

- 2026-09-09 (approx, exact date from Jira `updated`): Ad-hoc chat research (not yet a ticket file),
  found via production queries: 505 active products, 195 priced, 310 no-price (superseded by the
  2026-09-18 refresh above — dataset drifted by a few dozen rows).
- 2026-09-16: Resumed via "resume ticket AG-271" — checked live Jira per the reference memory's own
  instruction to verify before assuming "research only" — found the ticket's live description
  described a completely different baseline (726 lines, a already-fixed "manual price never read"
  bug, 2 heuristics with real numbers) that I could not reconcile with the schema. Reported the
  mismatch to the user rather than guessing; asked if they had the stakeholder deck.
- 2026-09-18: User said to just run a live production analysis directly. Re-verified 307 no-price
  rows (close to original 310), traced the "726" discrepancy to a disconnected legacy
  `inventory_stock` table (697 type='C' rows, no price column) — not reconciled, flagged as such.
  Then asked for a doc-format table of product names + prices mined from description — built via
  `mine_prices.js`/`mined_prices.json` → `Consumable_No_Price_Mined_From_Description.docx`.
- 2026-09-19: User caught that multi-price descriptions were only showing the last value — fixed via
  `mine_prices_v2.js` (collects all distinct keyword- and currency-tagged values), added S.No column,
  rebuilt and re-verified the docx.
- 2026-09-21: Resumed again ("we have completed the mining of price from description and now we will
  move to work on mining price with other methods") — this ticket file created retroactively at this
  point, per [[create-ticket-file-immediately-on-open]] applied as-soon-as-known since no ticket
  number/file existed for this recurring work until now.
