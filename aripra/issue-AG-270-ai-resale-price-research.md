---
name: issue-AG-270-ai-resale-price-research
description: AG-270 — resale-price estimation sub-task of AG-260; research + client docs, then a rule-based bracket formula actually built into the Overview tile and Task Card
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-09 — third round, now with real code built.** Started as a
research-only sub-task (closed 2026-09-07, no code), resumed 2026-09-09 for a follow-up comparison
round (still no code, closed again), then resumed a third time the same day and the rule-based
bracket option was actually implemented — Overview tile + Task Card, both live now. Not parked any
more for that specific approach; eBay/Gemini-based approaches (see comparison doc) remain parked
for a future resume if the rule-based estimate proves insufficient in practice.

**Ticket shape:** Jira Sub-task (not a Story), child of [[issue-AG-260-resold-resolution-path]] —
the direct answer to one of AG-260's own original open questions: "Is resale price always
staff-entered, or should the UI suggest a default (e.g. ~50% of PO price) staff can override?"

**Research findings:**
- Found directly relevant prior art already live in this codebase:
  `car-planet-backend/server/services/google-gemini/ai-dealer-price.service.ts`'s
  `suggestDealerPrice` — Gemini (`@google/generative-ai`), `temperature: 0`, prompt engineered to
  identify a part (part number as strongest identifier, then name/description/vehicle fitment) and
  return a bare number, null on failure/unrecognised (0 from Gemini = "couldn't identify").
  Confirmed via `.env` grep: only `GOOGLE_API_KEY`/`GOOGLE_GEMINI_MODAL` exist — no eBay API
  credential, no dedicated search API (SerpAPI/Bing/etc.) configured anywhere in this project.
- Identified the key factor that differs between dealer-price estimation (existing, live) and
  resale-price estimation (this ticket's ask): **condition**. Dealer price = always brand new,
  relatively stable/catalog-like, plausibly well-represented in an LLM's training data. Resale price
  = a used/salvage marketplace value, condition-dependent, no live grounding without an external
  search integration this project doesn't have. The part's **origin status**
  (`task_part.status` — Faulty vs Incorrect/Not Required) already captures the condition signal
  needed: Faulty → "spares or repair" tier (small fraction of working value); Incorrect/Not Required
  → physically unused, prices closer to a normal used/"new take-off" part. The original price paid
  (`poUnitPrice`/`invoiceUnitPrice`, already stored on `TaskPart`) is the natural scale anchor.
- Evaluated 3 approaches: (1) pure Gemini-prompt estimate, same mechanism as `suggestDealerPrice`,
  cheap to build, directionally useful but not eBay-precise; (2) web search + LLM extraction of
  comparable listings, more accurate in principle but a real new integration (no eBay/search API
  exists today) with added per-listing latency/cost; (3) historical-data-driven estimate from
  AutoGrid's own accumulating `resalePrice` data — most reliable long-term, **not viable yet**
  (confirmed earlier in this project session: only a handful of test rows with resale data exist
  locally, nowhere near enough to calibrate anything).
- **Recommendation: approach 1 (direct Gemini estimate) is the one worth building first**, once the
  user decides to resume this — reuses proven infrastructure, low cost, and the accuracy bar is
  appropriately low since the output is a staff-editable *suggested default*, not a final price (same
  framing AG-260's own open question originally used). Sketched a concrete prompt design (condition
  rules branching on origin status, same identification-rules structure as the existing
  `buildDealerPricePrompt`, same anchor-to-original-price pattern) — ready to hand to development
  once this is picked back up, not yet implemented.

**Deliverables produced (docs, not code):**
- `my-docs/projects/parts-and-consumable-inventory-4th-project/AI_Resale_Price_Estimation_Research.docx`
  — the full research write-up (background, resale-value factors, 3 approaches with honest
  tradeoffs, recommended-approach design, accuracy expectations, recommendation). Built the same way
  as the release-notes doc earlier this session — reused the same extracted-template + zip-rebuild
  scratchpad tooling (Word lacks a CLI here, no python available, hand-built a minimal ZIP writer in
  Node since neither `zip` nor an npm archiver package was available).
- `my-docs/projects/parts-and-consumable-inventory-4th-project/AI_Based_Price_Recovery.docx` — a
  second doc, originally titled/framed as "AI Dealer Price Data Requirements" (documenting exactly
  what inputs `suggestDealerPrice` needs and what causes a wrong/missing estimate — part number
  errors are the single riskiest failure mode, bare "engine" naming triggers uncapped major-assembly
  pricing, missing condition wording on assemblies defaults to brand-new pricing). **Renamed and
  reframed at the user's request** to "AI-Based Price Recovery — Estimating a Price for Parts With
  None on Record," generalizing the same technical content beyond just the dealer-price screen,
  explicitly tying it to [[issue-AG-269-consumables-price-lookup]]'s no-price consumables pool as
  "where else this could apply" — without overclaiming that's already built there. **Old filename
  (`AI_Dealer_Price_Data_Requirements.docx`) was NOT deleted** — attempted removal failed (file open
  elsewhere, "Device or resource busy"), then the user said not to bother removing it after all, so
  both files now sit in the project folder; the new one is the current/intended reference.

**2026-09-09 resume — 4th option added + comparison doc.** User resumed the ticket and asked
specifically whether eBay could be scraped/scrolled for resale comps. Answered directly (feasible
only via eBay's official Browse/Insights API, not scraping — ToS risk, fragility, no eBay
credentials configured in `.env` today; Insights API, which has actual sold prices rather than
inflated asking prices, requires an approval process outside our control). User then supplied their
own 3 options (eBay scrape, Gemini prompt, rule-based bracket off paid price/status/arrival date)
and asked for a full pros/cons/challenges/feasibility analysis plus any further options. Identified
the rule-based bracket approach as a genuinely new 4th option not covered in the original research
(cheap, deterministic, zero external dependency, uses only fields already on `TaskPart` —
`poUnitPrice`/`invoiceUnitPrice`, `status`, `partArrivedDate`/`firstFlaggedDate`) and recommended a
layered fallback chain (ship the rule-based bracket first as a floor, layer Gemini on top, treat
eBay's official API as a later accuracy upgrade, revisit AutoGrid's own historical `resalePrice`
data once enough real outcomes accumulate). Produced a new doc,
`my-docs/projects/parts-and-consumable-inventory-4th-project/AI_Resale_Price_Estimation_Options_Comparison.docx`
— a 5-column comparison table (Option/Feasibility/Pros/Cons-Risks/Key Challenges) covering eBay
scraping, eBay official API, Gemini prompt, rule-based brackets, and historical-data-driven (not
yet viable) — plus a "data already available" section and the recommendation. Built via the `docx`
npm package this time (installed fresh into a scratch dir) rather than hand-rolling OOXML/ZIP as
the original research doc required — much more reliable for a real table. Left the original
`AI_Resale_Price_Estimation_Research.docx` untouched (still open in Word, same file that blocked
deletion earlier) — this is a separate new file, not an edit to it.

**2026-09-09 — rule-based bracket actually built (Overview tile + Task Card).** Same day as the
comparison doc, user asked to implement option 3 from that doc — a deterministic formula, not an
AI/eBay call. Design: anchor = `invoiceUnitPrice ?? poUnitPrice`; bracket 50% for Faulty (salvage
tier), 80% for Incorrect/Not Required (physically unused); a further −5% per 30 days since
`partArrivedDate`, floored at 20% of anchor so it never suggests £0. **Placeholder percentages, not
validated against real sold prices** — explicitly flagged as tunable.
- New shared helper `computeSuggestedResalePrice(anchorPrice, status, partArrivedDate)` in
  `carplanet/.../non-conforming/non_conforming_tab.tsx`, exported next to the existing
  `computeDaysLeft` (same reuse precedent AG-284 established).
- **Requirement changed mid-flight** (screenshot-driven follow-up, same day): originally scoped to
  pre-fill the "Listed" action's `askingPriceInput` field in `part_note_panel.tsx` — before that was
  built, the user pivoted to two different surfaces instead: the Overview "Awaiting eBay Listing"
  tile, and the Task Card's info-icon tooltip. The `askingPriceInput` pre-fill was never built.
  - **Overview tile** (`overview_tab.tsx`, backend `getPartsResolutionBuckets` in
    `inventory.service.ts`) — the "Awaiting eBay Listing" tile's headline number flipped from
    `SUM(poUnitPrice)` (price paid) to a new `SUM(CASE...)` aggregate mirroring the bracket formula
    in raw SQL (`readyForSaleRecoverableValue`); price paid moved to an MUI `Tooltip` on hover. Only
    this one tile changed — the other 3 resolution-bucket tiles and the segmented bar/legend total
    still use paid price, kept apples-to-apples on purpose (not requested to change). No new join,
    no migration.
  - **Task Card** (`stock-details/task-v2/`) — `TaskPartsStatusBadge`'s existing info-icon tooltip
    (AG-274/AG-282's mechanism) gained a "Recoverable: £X" line, shown only when `status` is Faulty,
    Incorrect, or Not Required. Frontend `TaskPart` type (`types/task.ts`) gained
    `invoiceUnitPrice`/`partArrivedDate` — both already present in the wire response (`getTasks`
    loads full entities) but never modelled on the frontend before. `create_task_modal_new.tsx`
    passes the 3 new props through.
- `tsc` clean both repos, `next lint` clean on every changed frontend file.

**Related:** [[issue-AG-260-resold-resolution-path]] (parent ticket, the original open question this
answers), [[issue-AG-269-consumables-price-lookup]] (the "missing price" pool this could eventually
extend to).

---

## HISTORY

- 2026-09-05/07: User announced AG-270 as the next ticket to pick up. Fetched it from Jira, found it's
  a Sub-task of AG-260 (not a Story), explicitly research-only. Traced the existing
  `ai-dealer-price.service.ts` as directly relevant prior art before answering, rather than treating
  this as a from-scratch question. First-pass answer leaned cautious (flagged hallucination risk
  given the existing dealer-price service's own known prompt-contradiction issue); user pushed back
  ("what are the factors... can we get this via a gemini prompt... continue again") asking for a
  properly fleshed-out design rather than a risk-averse dismissal — re-did the analysis with actual
  prompt design detail (condition-tier branching on origin status, anchor-to-original-price), landing
  on a materially more actionable recommendation. User then asked for a client-shareable doc
  combining both rounds of findings — built via the docx-generation tooling reused from the earlier
  release-notes doc this session. Separately asked for a second doc specifically on what data the
  EXISTING dealer-price feature needs and what causes wrong/no estimates — built, then asked to
  rename/reframe it as "AI-based price recovery of parts missing a price today" rather than reading
  as dealer-price-specific — rebuilt under a new title achieving that without overclaiming what's
  actually live vs. what's a proposed generalization. Old file's removal was attempted (blocked, file
  in use) then explicitly waved off — no cleanup needed. User then said to mark AG-270 done and move
  to a new ticket, explicitly parking the actual AI-estimation build for a future resume, not
  abandoning it.
- 2026-09-09: User resumed a third time, asked to explain and get permission for implementing the
  rule-based bracket option (Overview tile / Task Card usage not yet specified). Explained the
  formula design, proposed pre-filling `part_note_panel.tsx`'s "Listed" asking-price field — got
  "yes go ahead" but before implementing, the user sent a screenshot-driven follow-up changing the
  target surfaces to the Overview tile + Task Card info tooltip instead (see NOW section). Traced
  both real data sources (`getPartsResolutionBuckets` for the tile, `getTasks`'s full-entity load
  for the Task Card) before proposing a revised plan, got a second "yes go ahead", built both
  surfaces + the shared `computeSuggestedResalePrice` helper. `tsc` clean both repos, `next lint`
  clean on every changed file. Gave commit messages for both repos on request, then user said to
  mark AG-270 done again and asked for `NEW_PARTS_AND_STOCK_INVENTORY.md` to be brought up to date
  for every ticket this session that hadn't been reflected there yet — added this ticket's own
  "Related ticket — AG-270" section + changelog entry, and separately fixed the doc's stale
  "Status: BUILT" wording on AG-277/278/284/258 (all had since been closed, doc never updated) plus
  a duplicated changelog header under AG-284 from an earlier edit.
