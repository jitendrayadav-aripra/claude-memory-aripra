---
name: issue-AG-270-ai-resale-price-research
description: AG-270 — research whether AI/LLM can estimate a part's resale price before it's listed, better than a flat percentage rule; the rule-based version got built, the AI-vs-rule comparison itself was never actually done
metadata:
  type: project
---

## NOW

**Status: DONE (for now), 2026-09-25 — memory-only close, Jira status untouched per
[[mark-as-done-means-memory-not-jira]].** Closed on explicit user instruction, despite the ticket's
own stated research question (AI/LLM vs. flat-rule resale-price estimation) never actually being
answered — see below, still true. Real-world context for why this was acceptable to close now:
[[issue-AG-299-external-parts-price-api|AG-299]]/AG-308 subsequently built a genuinely better answer
to "what should this part resell for" — a real eBay-sourced price (partner search, type-filtered
minimum, 10% under for a suggested listing price) that now takes priority over the flat-rule formula
wherever available, with the flat rule only as fallback. That's not the same as running the AI/LLM
comparison this ticket originally asked for, but it's a stronger real-world answer to the underlying
business need, which is presumably why closing this without ever doing that comparison was fine.

**Status before this close (resumed 2026-09-21):** Memory file did not actually exist despite `ARCHIVE.md` linking to
it — created retroactively now, from the archive-line summary + session history, before doing
anything further. Live Jira re-verified (unlike AG-271, this one matches memory closely — no
surprise mismatch).

**Live Jira description (verified 2026-09-21):** "Research whether AI can estimate a part's resale
price before it's listed. Right now staff have no price reference for a part once it moves to
'Waiting to be sold,' besides the original PO price it failed to sell at. This ticket already asks
whether the UI should suggest a default (e.g. ~50% of PO price). Check whether an AI/LLM estimate,
using the part's name, category, condition, and comparable listings on marketplaces like eBay, would
beat a flat percentage rule. This is research only, not a build commitment. Look for a viable
approach (e.g. web search plus LLM extraction of comparable prices, or a fine-tuned estimator) and
report back on feasibility and rough accuracy." Status: In Progress, sub-task of AG-255, reporter
Akash Robert, assignee Jitendra Yadav.

**What actually happened (from `ARCHIVE.md`'s summary, prior to this session):** research rounds
produced 3 client docs (`AI_Resale_Price_Estimation_Research.docx`, `AI_Based_Price_Recovery.docx`,
`AI_Resale_Price_Estimation_Options_Comparison.docx`). Then — instead of building the AI/LLM approach
the ticket actually asks about — **the flat/bracket percentage rule was built directly**:
`computeSuggestedResalePrice` (status-based bracket: 50% if Faulty, 80% if Incorrect/Not Required,
minus 5% per 30 days since arrival, floored at 20% of price paid). This now drives:
- The Overview tab's "Awaiting eBay Listing" tile's recoverable-value headline (later relabeled
  "(Estimated Recoverable Value)" this session, after the AI-mislabeling correction on
  [[issue-AG-295-parts-recovered-metric]] — see [[dont-claim-ai-without-verifying]]).
- The Task Card's info tooltip "Recoverable Price (AI estimated): £X" line — this label is ALSO
  wrong for the same reason (flagged, not yet fixed, tracked on AG-295's own memory file).

**The actual research question the ticket asks — AI/LLM vs. the flat rule — was never answered.**
No web-search-plus-LLM-extraction approach, and no comparison of its accuracy against the rule that
got built instead, has been done. This is the real open item if this ticket is to be considered
complete on its own stated terms, as opposed to "we built something adjacent and moved on."

**Related:** [[issue-AG-271-ai-price-estimation-research]] (same research family — AI-estimating a
*different* kind of missing price, the acquisition/cost price rather than resale price; same
calibration lesson applies — that ticket's Gemini calibration found a ~50-60% systematic
overestimation bias with no confidence/accuracy correlation, worth checking whether resale-price
estimation would show the same pattern if actually attempted).

---

## HISTORY

- 2026-09-03 to ~2026-09-09 (approx, from Jira `updated`): Original research + rule-based build,
  predates this memory store's per-ticket file convention for this specific ticket — only captured
  as an `ARCHIVE.md` summary line, no NOW/HISTORY detail retained from that period.
- 2026-09-21: Resumed via "resume ticket AG-270" — found the linked memory file never actually
  existed; created it now from the archive summary + live Jira re-verification, per
  [[create-ticket-file-immediately-on-open]] applied retroactively once noticed.
