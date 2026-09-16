---
name: dont-claim-ai-without-verifying
description: never leave or add an "AI"/"AI estimated"/"AI-generated" label on a value without verifying it's actually produced by an LLM/ML model, not a deterministic formula
metadata:
  type: feedback
---

Before adding, keeping, or reusing any user-facing "(AI)" / "AI estimated" / "AI-generated" label on
a value, verify it's actually produced by calling an LLM or ML model — not a fixed formula, however
sophisticated (percentage brackets, decay curves, thresholds, etc.).

**Why:** caught on [[issue-AG-295-parts-recovered-metric]] (2026-09-16) — the Overview tab's
"Awaiting eBay Listing" card was labeled "(AI estimated Recoverable price)", and I added a formula
tooltip explaining it without flagging that the figure is entirely deterministic
(`computeSuggestedResalePrice`: a status-based % bracket, minus 5% per 30 days, floored at 20% — no
model call anywhere in the chain). The same mislabeling was introduced earlier (AG-293/296) and
exists in at least one more place: `task_parts_status_badge.tsx`'s Task Card tooltip
("Recoverable Price (AI estimated)"). The user caught it, not me — I should have caught it while
writing the very tooltip that explains the formula.

**How to apply:** whenever I touch code near a value carrying an "AI" claim, trace where the number
actually comes from before accepting the label at face value. If it's a formula/heuristic with no
model call, say so — flag the mislabeling to the user rather than silently reinforcing it or writing
a comment/tooltip that repeats the wrong claim. This applies equally to genuinely AI-derived values
(e.g. `ai-dealer-price.service.ts`'s Gemini-based `suggestDealerPrice` is real and should keep an
"AI" label) — the point is to check, not to strip every "AI" label on sight.
