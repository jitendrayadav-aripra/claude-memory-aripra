---
name: issue-AG-298-drilldown-scrollbar-consumables-kpi-row
description: AG-298 — fix double vertical scrollbars + narrow width on the Overview drilldown side drawer, and make Consumables tab show all 5 KPI cards in one row like Overview's
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-17 in memory.** Fixed, `tsc`+`next lint` clean. Title/description
written to Jira via `editJiraIssue` (succeeded on first attempt). Jira ticket **status** left
untouched (Backlog), per [[mark-as-done-means-memory-not-jira]] — memory-only close unless told
otherwise.

**Root causes found + fixed** (`parts_inventory_drilldown_panel.tsx`):
- **Double vertical scrollbar**: the wrapper div had its own `overflow-y-auto` AND MRT's
  `enableStickyHeader: true` needs (and effectively gets) its own scrolling table container —
  two independent scrollable regions stacked. Fixed by making MRT's `muiTableContainerProps`
  (`maxHeight: "100%"`, `overflowY: "auto"`) the SOLE scrolling element; the wrapper div now only
  has `flex-1 min-h-0` (no overflow of its own). `min-h-0` overrides the flex item's default
  `min-height: auto`, which otherwise lets the wrapper grow to the table's full content height
  instead of respecting the available flex space — same root-cause family as the `min-w-0` fix
  from the AI-tooltip wrapping bug earlier this session. Also added `height: "100%"` +
  `display: flex/flexDirection: column` to `muiTablePaperProps` so the percentage chain down to
  `maxHeight: "100%"` actually resolves against a real pixel height, not an auto-height ancestor.
- **Drawer hidden behind the left sidebar**: the panel is `fixed w-[1500px] z-10`; the app's own
  nav sidebar is `fixed z-30` (`side_bar.tsx`), so wherever the drawer's left edge overlapped the
  sidebar (guaranteed on any screen narrower than 1500px, worse when the sidebar hover-expands to
  224px), the sidebar rendered on top. Fixed with `max-w-[95vw]` (keeps the drawer within the
  actual viewport instead of running off the left edge) and `z-40` (above the sidebar's `z-30`, so
  the drawer's own content always wins visually while open — same "wide overlay sits above
  persistent nav chrome" pattern used elsewhere for full-width panels).

**Consumables KPI row** (`consumables_tab.tsx`): `grid-cols-4` → `grid-cols-5` (both the real tile
grid and its loading-skeleton counterpart), matching Overview's fixed 5-column KPI row so the tiles
(today: Vehicle Specific / Office / Workshop / Other / Uncategorised — exactly 5) render on one row
instead of the 5th wrapping. Tile count is technically dynamic (canonical list + conditional
"Other"/extras), so a row with fewer than 5 will just leave empty column tracks — same behavior
Overview's own fixed grid would have with fewer items.

No backend change, no migration.

**Requirement, as given (2 bugs, both UI-only):**
1. The "Parts Pending Return or Resale" drilldown side drawer (`parts_inventory_drilldown_panel.tsx`,
   opened from any RESOLUTION_STAGE/AGEING_BUCKET card or bar) shows **two vertical scrollbars**
   (screenshot confirms — one at the far left edge, one at the far right, both redundant). The drawer
   is also **too narrow** — its left edge is hidden behind the app's own left sidebar, per the
   screenshot's green highlight showing content cut off/overlapping the sidebar.
2. Consumables tab's category KPI tiles (`consumables_tab.tsx`) render as **4 cards in row 1 + 1 card
   wrapping to row 2** (`grid-cols-4`), unlike Overview's 5 KPI tiles which all fit in a single row
   (`grid-cols-5`). Screenshot shows the 5th "Uncategorised" tile dropping to its own row. Requirement:
   match Overview's single-row-of-5 layout.

---

## HISTORY

- 2026-09-17: User initially said "new ticket: AG-295" — that number was already an existing, closed
  ticket ("Parts Recovered" metric, unrelated scope) confirmed via live Jira read. Flagged the
  mismatch; user corrected: "the actual new ticket is AG-298". Memory file created immediately per
  [[create-ticket-file-immediately-on-open]] once the real number was confirmed against Jira.
