---
name: issue-AG-260-resold-resolution-path
description: AG-260 — add a "Resold" resolution path (+ "mark as listed") for non-conforming parts stranded past their supplier return window
metadata:
  type: project
---

## NOW

**Status: BUILT, migration run 2026-09-01.** Final scope (after all doubts got answered): **Resold +
Listed only** — Refitted (AG-261) and Scrapped (AG-263) explicitly excluded from this pass (user
first said include them, then reversed that answer). Ownership question (doubt 7) resolved by
capturing `listedBy`/`resoldBy` (post-action attribution, mirroring `signedOffBy`) rather than
building any pre-action assignment mechanism.

**Where it landed (not where the ticket originally pointed):**
- Actions live in the **Part Returns Audit** tab (`Inventory > Part Returns Audit`,
  `non_conforming_tab.tsx` → `non_conforming_sign_off_modal.tsx` → `part_note_panel.tsx`'s Mode 1)
  — NOT the Leaderboard "Parts Oversight" module, NOT `part_return_note_modal.tsx`/Parts Delivered
  dashboard as earlier doubts had guessed. User picked this screen directly (shared screenshots) once
  Return Days Left was recognized as already computing the same return-window-expired eligibility.
- No Overview-tab bucket chart was built here — out of scope for this pass entirely (not just
  deferred to AG-256; never came up again after the mockup-review exchange).

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
- Migration: `1788269032635-add-resold-and-listed-fields-to-task-part.ts`, run by the user 2026-09-01.
- Comments were initially written far too long (precedent chains, repeated justification) — user
  flagged this explicitly; trimmed to one line + the one non-obvious fact everywhere, across both
  repos. Apply that standard to any future comment on this ticket's code.

**Not built / explicitly deferred:** the Overview bucket chart, Refitted, Scrapped, proactive
"suggest Resell" UI behavior (parked, "we'll decide later"), any pre-action ownership assignment.

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

**Next action:** none — feature shipped and migration run. If AG-256's Overview chart or AG-261/
AG-263 ever come back into scope, re-read this file's HISTORY for the full doubt-resolution trail
(chart ownership, naming conflict, modal-location correction) before restarting that work.

---

## HISTORY

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
