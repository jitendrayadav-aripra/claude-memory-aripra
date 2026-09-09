---
name: issue-AG-270-ai-resale-price-research
description: AG-270 — research sub-task of AG-260, can AI estimate a part's resale price before it's listed; no build, findings + client docs produced, parked for later implementation
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-09, re-parked.** Research-only sub-task, explicitly no build
commitment per its own ticket text — closed with **no code changes** both times, findings + client
docs delivered instead. **Parked again for a future resume when the user actually wants to
implement AI price estimation** — not abandoned, deliberately deferred, resumed once already
(2026-09-09) for a follow-up comparison round and expected to be resumed again.

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

**No code touched, no migration either round** — pure research + documentation ticket.

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
