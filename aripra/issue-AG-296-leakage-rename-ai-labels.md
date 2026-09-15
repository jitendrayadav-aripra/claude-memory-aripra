---
name: issue-AG-296-leakage-rename-ai-labels
description: AG-296 — rename "Parts Oversight" tab to "Part Leakage", add actual-price-paid to Task Card status hover, reword the "(AI)" labels to be self-explanatory everywhere they appear
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-15.** Title/description written to Jira via `editJiraIssue`.

User gave 3 requirements verbatim in chat, with 2 screenshots showing the exact current wording to
change:
1. Rename the "Parts Oversight" tab to "Part Leakage" (top-level leaderboard nav tab).
2. Add "Actual price paid" to the Task Card status badge's hover tooltip (currently missing
   entirely — the tooltip has Flagged/Resolved/Refitted/Recoverable(AI) lines but never shows what
   was actually paid for the part).
3. Reword the "(AI)" label everywhere it appears, since it's currently too terse to be
   self-explanatory:
   - Task Card tooltip: `Recoverable(AI): £X` → `Recoverable Price (AI estimated): £X`
   - Overview "Awaiting eBay Listing" tile's hover tooltip: `Price paid: £X` → `Actual price paid: £X`
   - Overview "Awaiting eBay Listing" tile's headline `(AI)` marker → `(AI estimated Recoverable
     price)`

Analysis in progress — need to find where the "Parts Oversight" tab label is actually defined
(likely the leaderboard tab bar, not just this module's own pages) before renaming, since a
mis-scoped rename could affect unrelated navigation.

**Related:** [[issue-AG-293-resale-recovery-retroactive-doc]] (built the original "(AI)" labels this
ticket is rewording), [[issue-AG-295-parts-recovered-metric]] (same tooltip areas, same session).

**User confirmed exact wording** ("Part Leakage", not "Parts Leakage") and 2 scope questions:
admin's "Team Permissions" checkbox (`teams_leaderboard_access.tsx:99`, also literally labeled
"Parts Oversight") — **explicitly excluded, left unchanged for now**; "Actual price paid" scoped the
same way the existing Recoverable line already is (Faulty/Incorrect/Not Required only), not shown on
every part status. Also asked whether the AG-295 "Parts Recovered" card should be renamed to
something with "Recoverable" in it for consistency — recommended against: "Recovered" (past,
definite — Resold/Refitted are actual realized outcomes) vs. "Recoverable" (future, estimated — the
AI guess) is a meaningful distinction worth keeping visually separate, especially now both sit on the
same widget. User agreed, left as "Parts Recovered."

**Built 2026-09-15, `tsc`+`next lint` clean.** No backend change, no migration — 3 frontend files:
- `leaderboard-tab.tsx` — renamed "Parts Oversight" → "Part Leakage" in both the breadcrumb label
  map (`tabLabels`) and the tab bar's own button text.
- `task_parts_status_badge.tsx` — new `actualPricePaid` value (same anchor as the existing
  `recoverablePrice`: `invoiceUnitPrice ?? poUnitPrice`), same eligibility scope. Added to the info
  icon's visibility condition too (since it's possible for a price to exist without
  `recoverablePrice` resolving, e.g. missing `partArrivedDate`). New tooltip line "Actual price
  paid: £X" alongside the reworded "Recoverable Price (AI estimated): £X" (was
  "Recoverable(AI): £X").
- `overview_tab.tsx` — "Awaiting eBay Listing" tile: tooltip "Price paid: £X" → "Actual price paid:
  £X"; headline marker "(AI)" → "(AI estimated Recoverable price)".

**2026-09-15 same-day follow-up — user asked to rename the admin permission label too after all**
(had explicitly excluded it earlier). `teams_leaderboard_access.tsx` — `labelText` "Parts Oversight"
→ "Part Leakage" (used the tab's exact wording for consistency, not the literal "parts Leakage" typed
in this follow-up message — flagged to the user). Permission KEY
(`LB_PARTS_OVERSIGHT_ACCESS`) deliberately left unchanged — that's a backend DB column, renaming it
would need a migration and is out of scope for a display-label change. `tsc`+`next lint` clean.

Title/description written into Jira via `editJiraIssue` (covers all 4 changes: tab rename + admin
label rename + Task Card price-paid addition + the 3 AI-label rewordings).

---

## HISTORY

- 2026-09-15: Ticket opened by user request ("new ticket: AG-296"), memory file created immediately
  per [[create-ticket-file-immediately-on-open]], before any analysis.
