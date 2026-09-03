---
name: issue-AG-260-resold-resolution-path
description: AG-260 — add a "Resold" resolution path (+ "mark as listed") for non-conforming parts stranded past their supplier return window
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-03**, per explicit user close-out. **Migration 3
(`1788426061602-add-refitted-scrapped-fields-to-task-part.ts`) run status was NOT re-checked before
closing** — same as this ticket's own migrations 1/2 being confirmed later on resume, verify before
assuming it's live if this area gets touched again. Everything below this line is the final build
record, kept for reference. The one follow-on item that came out of reviewing this ticket's own Part
Returns Audit pattern — Parts Inventory row-clicks redirecting to the vehicle Stock Details page
instead of opening the Task Card — was tracked and built as a **separate ticket, AG-272** (also
closed same day) rather than folded back into this one; see
[[issue-AG-272-parts-inventory-task-card-redirect]].

**Status: BUILT 2026-09-03, migration NOT yet run** — Refitted (AG-261) + Scrapped (AG-263) both
built in the same pass as the original Resold/Listed work. `tsc` clean both repos (backend + a
`next lint` pass on the 5 touched frontend files, all clean). **Migration 3
(`1788426061602-add-refitted-scrapped-fields-to-task-part.ts`) has NOT been run** — created via the
required `npm run migration:create` scaffold (real captured timestamp), not run by me per standing
rule ([[feedback_migration_commands]]) — hand back to the user to run when ready. Ownership question
(doubt 7) resolved by capturing `listedBy`/`resoldBy`/`refittedBy`/`scrappedBy` (post-action
attribution, mirroring `signedOffBy`) rather than building any pre-action assignment mechanism.

**What got built (2026-09-03):**
- Backend: 8 new `task_part` columns (migration 3, listed above) — `refittedDate(Text)`/`refitNote`/
  `refittedBy`, `scrappedDate(Text)`/`scrapNote`/`scrappedBy`. `markTaskPartAsRefitted` (blocks
  Faulty origin server-side, no listedDate requirement — two entry points) and
  `markTaskPartAsScrapped` (requires listedDate, no origin restriction) added to
  `inventory.service.ts`, both syncing `InventoryStock.status` (new enum values
  `InventoryStockStatus.REFITTED`/`SCRAPPED` in `inventory.type.ts` — `inventory_stock.status` is
  plain varchar, no migration needed there). New shared `TERMINAL_RESOLUTION_STATUSES = ["Resold",
  "Refitted", "Scrapped"]` constant replaces the old single-value `resoldVal` exclusion across all 6
  query sites that needed broadening (`buildNonConformingBaseQuery`'s virtual-option OR-bracket now
  has 7 buckets not 5, `taskPartsNonConformingFilterCounts` appends 2 more facet counts,
  `taskPartsDeliveredSection`, `getPartsWasteByMonth` + its `WASTE_MONTH` drilldown,
  `getPartsResolutionBuckets` + its `RESOLUTION_STAGE` drilldown). Controllers + PUT routes
  `/task-part/mark-as-refitted/:id` and `/mark-as-scrapped/:id` mirror the Resold pattern exactly
  (`req.user`, not `req.body.updatedBy`).
- Frontend: `NonConformingPart` type extended (8 new fields + 2 new `*ByUser` batch-fetch fields).
  Badge colours added across all 3 badge-style maps (`non_conforming_tab.tsx`,
  `non_conforming_sign_off_modal.tsx`, `part_note_panel.tsx`'s `BADGE_STYLE`): Refitted = emerald
  (`#047857`/`#ECFDF5`), Scrapped = stone/warm-gray (`#57534E`/`#F5F1EE`). `ALL_NC_STATUSES` in the
  filter panel extended with "Refitted"/"Scrapped".
- **`PartNotePanel` redesigned to an explicit action-toggle row** (the planned redesign, now built):
  replaced the old "infer the action from which field has content" heuristic (fine when only
  Listed/Resold existed) with a `selectedAction` state + a row of toggle-pill buttons showing
  whichever actions are currently eligible (`actionOptions`, computed from `isListed` +
  `isFaultyOrigin`) — pre-listing: Listed + Refitted (unless Faulty); post-listing: Resold + Refitted
  (unless Faulty) + Scrapped. Selecting a toggle shows that action's fields (Listed/Resold keep their
  dedicated inputs; Refitted/Scrapped have none — the shared comment textarea IS their note, same as
  Resold's `resaleNote` already was). Send performs whichever action is toggled on; no toggle
  selected = plain comment, unchanged from before. New `originStatus` prop threaded in from the
  sign-off modal (`part.status`) to compute the Faulty block. `isResolved` broadened to
  `Resold | Refitted | Scrapped` (any one is terminal).

**2026-09-03 Refitted/Scrapped scope, final decisions (before implementation):**
- Refitted does **not** attach a vehicle/task yet (deferred to later) — for now it's just a
  status+note action, same shape as Resold/Scrapped.
- Eligibility: Refitted is NOT shown for Faulty-origin parts (server-side blocked, mirrors the
  Resold-requires-Listed enforcement pattern); Refitted IS shown both before listing (from "Ready
  for sale") and after listing (from "Already on sale") — two entry points, unlike Resold/Scrapped.
  Resold and Scrapped are shown ONLY on already-Listed parts, for ALL origins (Faulty/Incorrect/Not
  Required), no origin restriction.
- No scrapped-price/expected-value column — "as right now client has no such requirement."
- **Separate dedicated columns** for Refitted/Scrapped (not shared/generic fields), same precedent
  as Listed/Resold: `refittedDate`/`refittedDateText`/`refitNote`/`refittedBy`,
  `scrappedDate`/`scrappedDateText`/`scrapNote`/`scrappedBy`. Decision reached after going back and
  forth on shared-vs-separate (see HISTORY) — landed on separate for consistency with the existing
  precedent, self-documenting schema value, and low real cost of a few extra columns on a table that
  already has 70+.
- **Note-requirement — RESOLVED 2026-09-03 (reversed from the earlier "undifferentiated" decision):**
  originally built with all three notes optional (user's first call — "we should do the same for all
  the status i.e RESOLD, REFITTED and SCRAPPED"), with the AG-263 differentiation parked as an open
  doubt. User then asked directly whether that doubt was a DB task or a coding task (clarifying they'd
  made it optional specifically to avoid a later migration) — confirmed it's coding-only: `scrap_note`
  is already a nullable `TEXT` column regardless of required-ness, so "required" is enforced purely at
  the app layer, no schema change needed either way. User then said **"so just make it required then
  as asked by the ticket"** — final answer: **Scrapped's note is REQUIRED, Resold/Refitted's notes
  stay optional** (differentiated after all, matching AG-263's "reason must be captured" text).
  Enforced in both layers: frontend (`sendDisabled`/`handleMarkAsScrapped` require
  `commentText.trim()`, placeholder reads "Reason for scrapping (required)...", red helper text under
  the Scrapped toggle) and backend (`markTaskPartAsScrapped` throws `StringError` if `scrapNote` is
  empty, mirroring the existing "must be listed first" guard). `NEW_PARTS_AND_STOCK_INVENTORY.md`'s
  open-doubt note should be updated to reflect this resolution too.
- **`InventoryStock.status` synced for ALL THREE (Resold, Refitted, Scrapped), not just
  Resold/Scrapped** — I initially recommended skipping the sync for Refitted (reasoning: the real
  stock-movement/destination-vehicle mechanism is deferred, so marking `InventoryStock.status =
  'Refitted'` now would be semantically premature). **User explicitly overrode this** with a
  practical operational reason, not just semantic preference: since this Refitted stock unit will
  **never re-enter the RFQ/procurement/check-in cycle again** (no new RFQ or PO is being created for
  it), the *original* `InventoryStock` row needs its status updated now regardless — otherwise it
  sits in limbo and could get swept up by some other process that expects stock rows to cycle
  normally through Used/Not Required/Incorrect. Accepted — sync all three now. Requires adding
  `InventoryStockStatus.REFITTED`/`SCRAPPED` to the backend enum (mirrors `RESOLD`, added earlier
  this ticket). **Revisit when the real Refit-to-new-vehicle mechanism eventually gets built** — at
  that point this stock row may need a genuine state transition (e.g. re-entering stock under a new
  task/vehicle) rather than just staying flagged `Refitted` forever; noted here so that future work
  doesn't assume `Refitted` is a true terminal InventoryStock state.

**Correction to this file's own earlier claim:** an older version of this NOW block said "no
Overview-tab bucket chart was built here — out of scope." That became wrong later the same
build — the user asked for it after all (2026-09-02/03) and it WAS built: `getPartsResolutionBuckets`
+ a `RESOLUTION_STAGE` drilldown case, rendered in `overview_tab.tsx` behind a
`SHOW_LEGACY_WASTE_CHART = false` flag (old "Waste run-rate" chart kept intact, just hidden — flip
the flag to bring it back). Read HISTORY below for the full sequence if resuming, don't trust a
NOW-block summary in isolation next time either.

**Where it landed (not where the ticket originally pointed):**
- Actions live in the **Part Returns Audit** tab (`Inventory > Part Returns Audit`,
  `non_conforming_tab.tsx` → `non_conforming_sign_off_modal.tsx` → `part_note_panel.tsx`'s Mode 1)
  — NOT the Leaderboard "Parts Oversight" module, NOT `part_return_note_modal.tsx`/Parts Delivered
  dashboard as earlier doubts had guessed. User picked this screen directly (shared screenshots) once
  Return Days Left was recognized as already computing the same return-window-expired eligibility.
- The Overview chart it feeds lives in `leaderboard/parts-inventory/overview/overview_tab.tsx` — a
  *different* module (AUT-3572's Leaderboard "Parts Oversight"), touched by this ticket only for
  this one chart, not repositioned there.

**Key implementation facts, in case of a future touch:**
- `task_part.status` is a real MySQL ENUM — required a `MODIFY COLUMN` migration (mirrors
  `1774915200000-Alter-task-part-status-add-incorrect-enum.ts`), not just a TS enum change.
  `inventory_stock.status` is a plain varchar — no migration needed there.
- `listedBy`/`resoldBy` are plain `int` columns, no `@ManyToOne` — confirmed via live DB that even
  `signedOffBy`/`partCheckedInBy` (which the entity decorates as relations) are plain int at the SQL
  level; `partActionedBy` is the exact same shape end-to-end. Resolved via batch-fetch in the list
  query, not a join (the query uses `.getManyAndCount()`, so a non-relation join can't hydrate).
- `markTaskPartAsListed`/`markTaskPartAsResold` use `LONDON_TIME_ZONE`, not `UTC_TIME_ZONE` —
  deliberately not copying `markTaskPartAsReturned`'s apparent inconsistency with the backend's own
  documented timezone rule.
- Controllers use `req.user` (reliable, matches `signOffTaskPart`), not `req.body.updatedBy` like
  `markTaskPartAsReturned`'s controller — that field is never actually sent by any frontend caller.
- Resold requires Listed first (server-side enforced) — confirmed via AG-261's authoritative
  end-state table, which the AG-260 ticket text itself didn't state explicitly.
- Migration 1: `1788269032635-add-resold-and-listed-fields-to-task-part.ts`, **run by the user
  2026-09-01** (added `Resold`/listing/resale columns, put `Resold` INTO the `status` ENUM — see
  migration 2, this got reverted days later).
- Comments were initially written far too long (precedent chains, repeated justification) — user
  flagged this explicitly; trimmed to one line + the one non-obvious fact everywhere, across both
  repos. Apply that standard to any future comment on this ticket's code.

**2026-09-02/03 follow-up — `resolutionStatus` redesign (user caught a real bug in the original design):**
`markTaskPartAsResold` originally overwrote `taskPart.status = 'Resold'`, permanently destroying the
origin flag (Faulty/Incorrect/Not Required) — the exact same class of bug flagged in
[[feedback_distinct_actor_timestamp_fields]], caught by the user asking "should Listed be an enum
too?" which led to realizing Resold already had this problem.
- New **`resolution_status`** column (real MySQL ENUM: `Listed`/`Resold`/`Refitted`/`Scrapped`,
  all 4 defined now even though only 2 are wired — user explicitly wants the full set defined once,
  not added value-by-value, since they consider this the final list). `status` now NEVER changes on
  Resold — stays the permanent origin flag forever, matching `firstFlaggedDate`'s existing "layer
  metadata, don't overwrite status" pattern that Listed already followed correctly from the start.
- Migration 2: `1788350681721-add-resolution-status-to-task-part.ts` — adds `resolution_status`,
  backfills it from the current `status`/`listed_date`, backfills `status → 'Not Required'` for
  rows that were `'Resold'` (arbitrary placeholder, fine per user — **confirmed local/staging only,
  not in production**), then removes `'Resold'` from `status`'s ENUM entirely.
  **Migration-run status: CONFIRMED run by the user (2026-09-03 resume check).** Both migrations
  for this ticket are now live.
- 5 query sites needed a `resolutionStatus != 'Resold'` exclusion added (status alone no longer
  implies "still unresolved"): `buildNonConformingBaseQuery` (Part Returns Audit), the new
  `getPartsResolutionBuckets` + its `RESOLUTION_STAGE` drilldown case, the hidden legacy waste chart
  + its `WASTE_MONTH` drilldown case, and `taskPartsDeliveredSection` (Parts Delivered dashboard).
- **UI iterated twice, ended back near the original design** — worth reading before touching this
  again: (1) tried a genuinely separate "Resolution Status" table column + independent AND-combined
  filter section; user caught two real problems — the column is empty for the vast majority of rows
  (most parts never resolve), and the AND-combined filter created a trap where checking "Resold"
  alone against the default Part Status filter (`Returned` only) silently returned zero rows. (2)
  Reverted to: single Status badge (`resolutionStatus ?? status`) with a small `InfoOutlined` hover
  icon on resolved rows showing "Originally: {status}"; `Listed`/`Resold` merged back as virtual
  OR-combined options in the *same* Part Status checklist (exact same mechanism as the pre-existing
  "Signed Off" virtual option) — not a separate filter, so selecting "Resold" alone always works.
- **`PartNotePanel`'s Send button UX also fixed** (2026-09-03): originally had a separate "Mark as
  Listed"/"Mark as Resold" button alongside the general "Send" button — confusing, two buttons doing
  unrelated things right next to each other. Fixed by merging: Send's label/behavior now switches to
  the Listed/Resold action once those specific fields have content, otherwise behaves as a normal
  comment Send — no toggle needed, since Sign Off and Listed/Resold can never apply to the same part
  (mutually exclusive by `isNotReturned`), so there's no ambiguity a toggle would need to resolve.

**Not built / explicitly deferred:** Refitted's vehicle/task attachment (destination-vehicle
linking — real stock movement stays TODO), proactive "suggest Resell" UI behavior (parked, "we'll
decide later"), any pre-action ownership assignment. Refitted/Scrapped's status+note action itself
is now in progress, not deferred — see the 2026-09-03 block above.

**AG-258 (Parts Inventory — People tab, also In Progress)** — worth knowing: it's the client-side
spec mirroring AUT-3572's People tab. Confirms "Workshop manager" = `Vehicle.mechOwner` via the same
join already built there; independently cross-checked 21 real vehicles against a known-good export,
**80% accuracy match** — a different measurement than AUT-3572's sparsity finding (2/28,591 vehicles
have `mechOwner` set at all), not a contradiction: rarely populated, but trustworthy where it is.
Flags a real gotcha — **`Vehicle.mechCommentOwner`** is a separate, unrelated decoy field bound to a
different UI component; confirmed via grep that AUT-3572's code never touched it, only real
`mechOwner` — no bug there.

**Jira:** https://aripra.atlassian.net/browse/AG-260 (Story, Backlog, assigned to the user, Medium
priority, reporter Akash Robert). Fetched via the now-connected **Atlassian Rovo MCP** connector
(`claude.ai Atlassian Rovo` / `plugin:atlassian:atlassian`, cloudId `761f8a59-91ff-493b-ab27-a38827186c27`,
also usable as `aripra.atlassian.net`) — first successful direct Jira fetch this project has had;
previously blocked (see [[issue-AUT-3572-parts-consumables-inventory-module]]'s history). The
connector needed authenticating in an interactive `claude` CLI terminal session (`/mcp`) — the VS
Code extension session doesn't share that auth automatically; reconnected once, should persist for
future direct fetches without re-doing OAuth (unless the harness treats it as a fresh session).

**Related tickets** (Jira, all fetched and read this session):
- **AG-255** (Epic) — "Parts Inventory — Leakage & Accountability Dashboard," the client-side twin
  of [[issue-AUT-3572-parts-consumables-inventory-module]]. Epic-level open question explicitly
  flags AG-256-vs-AG-260 bucket-UI ownership as unresolved.
- **AG-256** — Overview tab (KPI tiles + charts) rework. Jira status **"Ready for QA"**, but this
  codebase's `overview_tab.tsx` still has the OLD "waste-by-month"/"Written off" chart — none of
  AG-256's rename ("Flagged Parts Pending Resolution") or the 4-bucket UI has landed on `beta3.0`
  yet. Real discrepancy, not yet explained — see doubt (2).
- **AG-257** — not yet read (Alert Checks tab, inferred from AG-255's epic description; not fetched).
- **AG-261** — "Refitted" exit (refit to a different vehicle, no money changes hands). Sibling to
  this ticket; not yet read in detail. Business rule: legal from Not Required/Incorrect origin,
  BLOCKED from Faulty origin, must be server-side enforced — owned by AG-261, just noted inside
  AG-260 for context.
- **AG-263** — "Scrapped" exit (permanent write-off, "some of them are worthless" — Ahmad,
  WhatsApp 27 Aug 2026). Not yet read in detail.
- **AG-264** — blocks AG-256 (invoice price not used in reporting); unrelated to AG-260 directly.

**What AG-260 requires (full detail in ticket body, summarized):**
Faulty/Incorrect/Not Required parts past their supplier return window (`Supplier.returnWindowDays`)
currently have no closing move — 1,224 parts / £59,091.68 stranded on the live DB, avg ~300 days
old. Locked resolution model (client, 27 Aug 2026): every flagged part computes into one bucket —
**Waiting to be returned** (in-window) / **Waiting to be sold** (past window, split **Ready for
sale** / **Already on sale**) / **Unknown** (`returnWindowDays IS NULL`, a data-quality gap, dashed
border not a peer category). This ticket owns two exits from "waiting to be sold": **Resold** and
**"mark as listed"** (Ready for sale → Already on sale, metadata-only, doesn't change
`TaskPart.status`). Refitted is AG-261's exit, not this ticket's.

**Next action on resume:** first confirm whether migration 2
(`1788350681721-add-resolution-status-to-task-part.ts`) has actually been run — not confirmed in
this session. Then, if AG-261/AG-263 ever come into scope, the `resolution_status` ENUM already has
`Refitted`/`Scrapped` defined (schema-ready, zero migration needed) — just wire the write paths and
UI. If AG-256's Overview chart work overlaps with the one already built here
(`getPartsResolutionBuckets`), reconcile rather than duplicate.

---

## HISTORY

- 2026-09-03: User asked to pause AG-260 and move to a new ticket — marked DONE (for now). Before
  that, three follow-up rounds happened after the initial build (see NOW for the technical summary):
  (1) 2026-09-02, user asked for the Overview "Flagged Parts Pending Resolution" chart after all
  (reversing the earlier "out of scope" close) — built `getPartsResolutionBuckets` +
  `RESOLUTION_STAGE` drilldown, rendered behind a `SHOW_LEGACY_WASTE_CHART` flag so the old chart
  stays recoverable; confirmed no new npm dependency needed (segmented bar is plain divs), which the
  user then turned into a standing rule ([[no-library-install-without-permission]]). (2) User asked
  whether Listed should become an enum too — led to catching that Resold already had exactly that bug
  (overwriting `status`, losing the origin flag); agreed, built a separate `resolution_status` column
  instead (migration 2), with a full round of "should this be a real ENUM or VARCHAR" discussion that
  landed on ENUM once the user confirmed all 4 values (Listed/Resold/Refitted/Scrapped) are meant to
  be defined up front as a final set, not grown one value at a time. Also debated and resolved
  whether to backfill the 2 pre-existing local `status='Resold'` rows to `NULL` or to a placeholder
  status (`Not Required`, chosen — confirmed local/staging only, no real data risk). (3) The UI for
  showing both status + resolutionStatus together went through a full cycle: built a separate
  "Resolution Status" table column + filter section → user caught it was mostly-empty (most parts
  never resolve) and created a filter trap (AND-combined against a restrictive default Part Status
  filter meant checking "Resold" alone often returned zero rows) → reverted to a single badge +
  hover-info-icon in the table, and merged Listed/Resold back into the existing Part Status checklist
  as virtual OR-combined options (same mechanism as "Signed Off"). Separately fixed `PartNotePanel`'s
  UX — a standalone "Mark as Listed/Resold" button sat right next to the general "Send" button,
  confusing which to click; merged them so Send's behavior follows whether the action-specific fields
  have content. Full detail — including that migration 2's run status is NOT confirmed — is in NOW.
- 2026-09-01 (3rd entry): User shared a mockup of the Overview 4-bucket chart, revealing it's being
  actively built already (reads as Akash) — resolved doubt 1 (chart out of scope for this pass). Also
  surfaced a naming conflict (mockup says "Unresolved Parts Value," AG-256's ticket mandates verbatim
  "Flagged Parts Pending Resolution") and a real dependency (the chart's Ready-for-sale/Already-on-sale
  split needs AG-260's `listedDate` field to exist first). User then answered all remaining doubts:
  build both frontend+backend regardless of Jira ownership metadata; explained the Listed→Resold
  lifecycle in plain terms (corrected via AG-261's authoritative end-state table: Resold requires
  Listed first, not a direct exit from Ready-for-sale); resale price staff-entered only (reconfirmed);
  proactive-suggest parked; **Refitted/Scrapped scope reversed twice** (said include, then said
  exclude — final answer: AG-260 only); ownership resolved via post-action attribution, not
  pre-assignment. Shared screenshots of the actual "Part Returns Audit" tab, which turned out to be a
  better fit than any previously-guessed location — has the exact right filter population and a
  "Return Days Left" column already computing the eligibility logic AG-260 needed. Read the project's
  `project-lessons.md` before building (user's explicit instruction) — this caught two things that
  would have been wrong: (1) `listedBy`/`resoldBy` must be plain int, not `@ManyToOne`, confirmed via
  live DB schema query (`signed_off_by`/`part_checked_in_by` are plain int at the SQL level despite
  relation decorators in the entity; `part_actioned_by` is the exact same shape); (2) `task_part.status`
  is a real MySQL ENUM requiring a `MODIFY COLUMN` migration, found the exact precedent
  (`1774915200000-Alter-task-part-status-add-incorrect-enum.ts`). Also caught mid-build: the ticket's
  own "mirror markTaskPartAsReturned" instruction would have propagated two real bugs if followed
  literally — UTC_TIME_ZONE instead of LONDON_TIME_ZONE, and `req.body.updatedBy` instead of the
  reliable `req.user` (the former is silently never sent by any frontend caller). Built both repos,
  `tsc`/`eslint` clean on both. User then flagged the code comments as far too long (precedent chains,
  repeated justification) — trimmed every AG-260 comment across both repos to one line + the single
  non-obvious fact, re-verified `tsc` clean after. User ran the migration
  (`1788269032635-add-resold-and-listed-fields-to-task-part.ts`, decodes to 2026-09-01 13:23:52 UTC /
  14:23:52 BST) and confirmed. Updated `NEW_PARTS_AND_STOCK_INVENTORY.md` (AUT-3572's living doc) with
  a cross-referencing "Related ticket" section, since this ticket touches a different screen than that
  doc's main scope.
- 2026-09-01: User asked to re-analyze — client had edited the Jira ticket. Re-fetched AG-260: status
  Backlog → In Progress; only the "Ownership / accountability" section's text changed (see NOW),
  citing new sibling ticket AG-258 (fetched for context — People tab, confirms `Vehicle.mechOwner`
  with an 80%-accuracy live cross-check, flags the `mechCommentOwner` decoy field). Re-verified
  nothing material had landed in either repo since 2026-08-31 (one unrelated 1-line commit). No new
  build requirement found — added doubt 7 (mechOwner-style ownership for "Ready for sale," in-scope
  for this ticket or not) on top of the original 6, still all unanswered. No code touched.
- 2026-08-31: Ticket opened by user request ("new ticket: AG-260"). First successful direct Jira
  fetch on this project — Atlassian Rovo MCP connector was not connected in this VS Code extension
  session initially (`/mcp` showed 0/2 connected) despite being already authenticated in a separate
  interactive `claude` CLI terminal session (shown via screenshot: `claude.ai Atlassian Rovo` +
  `plugin:atlassian:atlassian`, both connected, 36 tools each) — user reconnected it mid-session
  (`/mcp` in the CLI terminal went 0→2→3 connected), after which the MCP tools became available
  here too. Fetched AG-260 + its two most load-bearing linked tickets (AG-255 epic, AG-256) via
  `getJiraIssue`. Cross-checked every concrete claim in the ticket against the actual codebase
  (grep + targeted reads, not guesses) and found two real discrepancies between the ticket text and
  current code state (see doubts 2 and 3 above) plus one genuine unresolved-in-Jira-itself ownership
  question (doubt 1). Confirmed `parts_inventory_tab.tsx` (the file the user pointed me at) needs no
  changes — it's just the sub-tab router. Presented full understanding + 7 doubts to the user per
  their explicit "ask doubts, then ask permission before applying" instruction; user then paused the
  ticket for an urgent unrelated module before answering. No code touched.
