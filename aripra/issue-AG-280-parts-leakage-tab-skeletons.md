---
name: issue-AG-280-parts-leakage-tab-skeletons
description: AG-280 — add loading skeletons to all Parts Leakage tabs (Overview, People, Alert Checks, Consumables), currently no loading state shown while data fetches
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-16 in memory.** Code built, tsc+lint clean. **Jira ticket status
NOT touched further** — left at "In Review/QA" (see below for how it got there). Per
[[mark-as-done-means-memory-not-jira]], "mark as done" closes this in memory only; the Jira
ticket's actual status is a separate, explicit ask every time.

Note: earlier in this same session, before that lesson was established, I *did* call
`transitionJiraIssue` on this ticket in response to "mark as done" (Reported → In Review/QA),
which side-effect-reassigned it to Shivansh Shukla (the original reporter) — this was a live
mistake that prompted the user to state the rule. Left as-is unless the user asks me to do
anything further with the Jira ticket.

This was a **real pre-existing bug ticket** (filed 2026-09-08 by Shivansh Shukla, "AG-257 -> After
landed on this page so there should be a proper loader or skeleton loader", with a screenshot) — NOT
a retroactive/blank ticket like AG-297, so no title/description needed to be written.

Requirement: none of the 4 Parts Leakage tabs (Overview, People, Alert Checks, Consumables) showed a
skeleton while their data was loading. Found the codebase's existing MUI `Skeleton` pattern
(`advert_stat_cards.tsx`/`advert_need_attention_list.tsx` in stock-details — per-section `Skeleton`
sized to match real content, same card chrome) and MRT's own built-in `state.showSkeletons` for
table-shaped loading (no shared generic skeleton wrapper exists in this codebase).

**Built:**
- `overview_tab.tsx` — replaced the whole-tab `"Loading..."` with per-section `Skeleton`s (5 KPI
  cards + 2 chart panels) inside the same card chrome.
- `consumables_tab.tsx` — same treatment for the 4 category tiles; stock table wired to MRT
  `showSkeletons`.
- `people_tab.tsx` (pivot table), `people_person_table.tsx`, `parts_inventory_alert_table.tsx`,
  `parts_inventory_ready_cars_table.tsx` — wired `state: { showSkeletons: isPending }` (or the
  pivot's own plain `isLoading`) into their existing `useMaterialReactTable` configs. Used
  `isPending`, not `isFetching`, specifically because `isFetching` also fires on scroll-triggered
  `fetchNextPage` calls and would wrongly flash skeletons over already-loaded rows.

No backend changes, no migration. `tsc --noEmit` clean, `next lint` clean on all 6 changed files.

---

## HISTORY

- 2026-09-16: Ticket opened by user request ("new ticket is AG-280"), memory file created immediately
  per [[create-ticket-file-immediately-on-open]], before any analysis.
