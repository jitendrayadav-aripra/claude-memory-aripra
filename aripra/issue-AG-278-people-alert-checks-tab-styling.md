---
name: issue-AG-278-people-alert-checks-tab-styling
description: AG-278 — People tab role toggle restyled + Alert Checks converted from stacked cards to a tab bar, matching QC module's PDI toggle / Exterior-Interior tab patterns
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-11.** Originally closed 2026-09-08 as pure UI restructuring, then
bounced back 2026-09-11 for a real regression (all 4 Overview tiles landing on the wrong Alert
Checks tab) — regression fixed same day, `tsc`+`next lint` clean, re-closed by explicit
instruction.

**Source:** two screenshots from the QC/PDI module (Prep > Inspection Dashboard > Pre Delivery
Inspection's "Pending PDI / Completed PDI" toggle, and a Cosmetic-Inspection-style "Exterior [0] /
Interior [0]" tab control), given as the exact visual reference to match — no existing-component
hunt needed, styled directly to match the screenshots (an earlier attempt to grep/agent-search for
the exact source components was explicitly stopped by the user as overkill for a styling task).

**What got built, in `carplanet/src/app/dashboard/leaderboard/parts-inventory/`:**
- `people/people_tab.tsx` — the `PO created by`/`Workshop manager`/`Technician` role switcher
  changed from a row of individually-outlined chips to a seamless segmented control.
- `alert-checks/alert_checks_tab.tsx` — rewritten from a flat stack of 4 always-visible table cards
  (Late to arrive / Received >2 days / Nothing fitted / Car gone) to a tab bar (name + live count per
  tab, both always visible). Only the active tab's table is visually shown; the other 3 stay mounted
  with `display:none` rather than being unmounted, specifically so all 4 tables' counts populate and
  stay current immediately (matching the reference design, which shows every tab's count up front,
  not only after switching to it) — this was a deliberate call, not an oversight; it doesn't change
  network cost vs. before, since all 4 were already fetching concurrently in the old stacked layout,
  just now only one is rendered visually at a time.

**2026-09-08 correction — exact Figma tokens supplied, first pass had both the colours and the
structure wrong.** First build used a padded floating-pill look (outer `bg-[#F2F2F6] rounded-full
p-1` container, inner active segment `bg-[#111827]` near-black, inactive = transparent/plain text) —
guessed from the screenshots' general appearance rather than exact values, since an earlier attempt
to hunt down the literal source component elsewhere in the app was explicitly stopped by the user as
overkill. User then supplied the real design tokens directly: active segment
`background: var(--Text-Subtext, #45556C)`, inactive segment
`background: var(--Surface-Border-Light, #F2F2F6)`, and — the structurally important part — rounding
applied per-segment to only ONE side (`border-*-right-radius` on the active example, `border-*-left-
radius` on the inactive example), which describes a **seamless two-tone bar with no gap between
segments**, rounded only at the two outer ends of the whole control, not a floating pill inside a
padded track. Rebuilt both toggles: removed the outer padding/background wrapper, gave each segment
button a flat `h-8 px-5` (no vertical padding — height is the fixed dimension, content centers via
line-height), and only the first segment gets `rounded-l-full` / only the last gets `rounded-r-full`,
generalizing the 2-segment Figma example correctly to the People tab's 3 segments and Alert Checks'
4 segments (a case the literal 2-segment example didn't directly cover — middle segments get no
rounding at all). Explicitly did NOT hardcode the given `121px`/`105px` widths — those were Figma's
auto-width for that specific instance's own text ("Exterior"/"Pending PDI"), not meant as fixed
values for this project's different, longer labels ("PO created by"); kept width intrinsic to each
label's content instead, flagged this assumption to the user rather than silently deciding it.
- `alert-checks/parts_inventory_alert_table.tsx` and
  `alert-checks/parts_inventory_ready_cars_table.tsx` — both gained an optional
  `onTotalChange?: (total: number) => void` prop, fired via a `useEffect` keyed on their own derived
  `total` (from the React Query page's `count`), so each table reports its live count up to the new
  parent tab bar without changing how each table fetches/paginates internally.

**2026-09-08 follow-up — dropped the "N ·" number prefix only, kept the titles themselves.** User
caught that the 3rd table (`PartsInventoryReadyCarsTable`) had no "3 ·" number/title showing, unlike
the other 3. Root cause: `PartsInventoryReadyCarsTable` is a standalone component with its own
hardcoded internal `<h2>` heading (no `title` prop at all), unlike `PartsInventoryAlertTable` (shared
across late/breach/gone, taking `title` as an external prop) — the "N ·" numbering was hand-typed
only into the 3 prop strings passed from `alert_checks_tab.tsx`, and nobody ever added a matching
"3 ·" prefix to the 4th component's hardcoded string. Pre-existing gap from the old stacked-4-cards
layout, not introduced by AG-278's tab-bar rewrite.

Asked the user which fix they wanted — add the missing "3 ·", or drop the numbering from all 4 since
the tab bar now shows each name anyway. User said drop it. **First attempt over-corrected**: removed
the entire `<h2>{title}</h2>` + count-badge header row from both table components (not just the
number), which the user then flagged — "you have just removed the whole title... I was asking to
drop only the number and not the whole title." Reverted to keep the title + count-badge header row
in both components exactly as it was, just with the leading "N · " stripped from each of the 3
`title` strings passed in `alert_checks_tab.tsx` ("Late to arrive", "Received >2 days, not used —
cars still in stock", "Car gone, parts still on the shelf") and `PartsInventoryReadyCarsTable`'s own
hardcoded title left as "All parts arrived, car waiting — nothing fitted" (it never had a number to
begin with). `tsc`+`next lint` clean after the correction.

**2026-09-11 — bounced back, real regression in this ticket's own build.** Since Alert Checks
moved from 4 stacked tables to a single-tab-at-a-time bar, all 4 Overview KPI tiles share one
identical handler, `goToAlertChecks` (`overview_tab.tsx:64-65`), which routes to
`?tab=PARTS_INVENTORY&sub=ALERT_CHECKS` with no worklist identifier — confirmed this is a real,
current bug, not hypothetical. `AlertChecksTab` always initializes `activeTab` to `"late"`
(`alert_checks_tab.tsx:21`) and never reads a URL param to override it. Net effect: all 4 tiles
land on "Late to arrive" regardless of which one was clicked.

**Confirmed the exact tile→tab mapping the fix needs** (matches the ticket's own AC): Received but
unfitted → `ready` ("Nothing fitted"); Parts arrived on sold cars → `gone` ("Car gone"); Overdue
fitting → `breach` ("Received >2 days"); Awaiting delivery → `late` ("Late to arrive"). Each tile's
own stat semantically matches its target worklist exactly (e.g. "awaiting delivery"/on-order parts
are the ones still in the late/no-ETA pipeline).

**Confirmed the exact URL-param pattern to reuse, not invent new**:
`parts_inventory_tab.tsx` already does this same thing one level up — `useSearchParams()` +
`useEffect` reading `sub=` to set `activeSubTab`, and `router.push` building the URL with that
param. Plan: extend `goToAlertChecks` to take a tile-specific key and append `&alertTab=<key>`;
`AlertChecksTab` reads it the same way `parts_inventory_tab.tsx` reads `sub=`, falling back to the
existing `"late"` default when absent (so the main tab bar's plain navigation stays unaffected, per
the ticket's own 2nd AC).

**2026-09-11 — regression fix built, `tsc`+`next lint` clean.**
`overview/overview_tab.tsx`: `goToAlertChecks` now takes `alertTab: "late" | "breach" | "ready" |
"gone"` and appends `&alertTab=${alertTab}` to the pushed URL; all 4 KPI tiles updated to pass their
matching key (Received but unfitted → `ready`, Parts arrived on sold cars → `gone`, Overdue fitting
(>2 days) → `breach`, Awaiting delivery → `late`) — confirmed via grep that no tile still calls the
old parameterless form. `alert-checks/alert_checks_tab.tsx`: added `useSearchParams()` +
`useEffect` reading `alertTab`, validated against an `ALERT_TAB_KEYS` array before calling
`setActiveTab`; invalid/absent param leaves the existing `useState<AlertTabKey>("late")` default
untouched, satisfying the ticket's 2nd AC (plain nav via the main sub-tab bar unaffected). No
backend involvement — pure frontend regression.

**Related:** [[issue-AUT-3572-parts-consumables-inventory-module]] (the module this lives in).

---

## HISTORY

- 2026-09-08: User gave the ticket number (AG-278, no description in Jira yet) plus two annotated
  screenshots and asked for analysis + direct implementation + a ticket description afterward (no
  "ask permission first" gate this time, unlike every other ticket this session — explicit
  instruction was "apply the changes" as part of the same ask). First instinct was to spawn an
  Explore agent hunting for the exact source components behind both reference screenshots elsewhere
  in the app (to copy their literal classNames) — user interrupted and pushed back ("what are you
  searching so deep for... I just told you for the UI changes"), correctly reading this as overkill
  for a styling-match task. Corrected course: built both patterns directly from the screenshots'
  visual description, using this module's own established color/spacing conventions (`#1B55EC`
  blue, `#0F172B`/`#111827` dark, `#45556C` muted text, `#F2F2F6` borders/backgrounds — all already
  used throughout `parts-inventory/`) rather than reverse-engineering an unrelated module's exact
  markup. Built both changes, `tsc`+`next lint` clean, updated the living project doc.
