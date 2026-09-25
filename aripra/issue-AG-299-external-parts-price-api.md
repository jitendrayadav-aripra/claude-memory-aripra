---
name: issue-AG-299-external-parts-price-api
description: AG-299 — build 2 external-facing API endpoints (bulk + single-part) exposing "Awaiting eBay Listing" part details, so an external server can pull them and run its own price extraction; sub-task of AG-260
metadata:
  type: project
---

## NOW

**Status: DONE (for now), 2026-09-25 — memory-only close, Jira status untouched per
[[mark-as-done-means-memory-not-jira]].** Final shape: a daily-scheduled eBay price-search queue
(not the original immediate-trigger design — see the full pivot history below), with the real
eBay-sourced price (10% under the cheapest matching listing) shown on the Task Card tooltip, the
Part Returns Audit "Mark as Listed" popup, and folded into the Overview tile's "Estimated
Recoverable Value" total in place of the AI-estimate formula wherever a part has one. Documented in
`my-docs/projects/parts-and-consumable-inventory-4th-project/NEW_PARTS_AND_STOCK_INVENTORY.md`'s own
"Related ticket — AG-299" section (added 2026-09-25).

**Still genuinely open, not just deferred-and-forgotten:**
- Staging/production migrations: written, NOT run anywhere except locally by the user — needs
  running on each environment before this works there.
- The partner's actual result-submission payload does not match our documented
  `{ "ebayResult": ... }` wrapper contract (they send it flat) — 2 fix options were given (ask them
  to wrap it, or make our endpoint tolerant of both shapes); **no decision was made**, still pending.
- A staging deployment-pipeline issue (system-crontab `auto-deploy.sh`, PM2 PATH problem) was
  discovered while testing this — unrelated to our code, being worked on by a teammate (Hemant
  Devde), not something this ticket owns or fixes.
- Production-target branch (`AKASH/Feature-AG-299-2-...`) was audited file-by-file and confirmed to
  have every required change (stall timeout, single-digit exclusion, correct 2 AM cron time,
  no-staging gate) — but has not actually been pushed/deployed to production as of this writing.

**Original status line, kept for history:** bulk endpoint built 2026-09-21, `tsc` clean, not
deployed. Single-part endpoint explicitly deferred — user said "for now let's create a bulk api
only."

**Major design correction mid-session — the auth plan changed for the better.** Original plan (a
new dedicated shared secret modeled on the ETC webhook) was proposed and approved, but before
writing the auth code, discovered this codebase already has a COMPLETE, production-grade external
API-key system (`middlewares/api-key-auth.middleware.ts`, `/api/external` namespace,
per-client route-pattern allowlisting, admin UI at Admin → "External API Clients" tile, gated by
`API_CLIENT_MANAGEMENT_ACCESS`). Flagged this to the user immediately rather than silently swapping
approaches — this is strictly better than a shared secret (per-endpoint scoped grants, revocable
from the UI with no deploy, built-in audit/analytics per client) and is the established convention
for exactly this kind of partner integration (existing precedent: Cardaddy's partner API at
`/api/external/cardaddy/...`). No new secret/env var needed at all.

**What got built (renamed 2026-09-21, see below):**
- `services/inventory/ebay-listing-export.service.ts` — `getPartsAwaitingEbayListingExport()`.
  Deliberately kept OUT of the already-7000+-line `inventory.service.ts` (external-integration
  concern, not an internal feature). WHERE conditions copied verbatim from
  `getPartsInventoryDrilldown`'s own `RESOLUTION_STAGE === "READY_FOR_SALE"` case (flagged
  Faulty/Incorrect/Not Required + not a terminal resolution status + return window expired +
  `resolutionStatus IS NULL`) — re-verified line-by-line against that exact code before considering
  it done, so this export can never silently drift from what the Overview tile shows.
- `controllers/ebay-listing-export.controller.ts` — thin handler, `IApiKeyRequest` typed.
- `routes/ebay-listing-export/ebay-listing-export.route.ts` — mirrors `routes/cardaddy/cardaddy.route.ts`'s
  structure exactly. Mounted onto `routes/api-external/index.router.ts`'s `externalRouter` at
  `/ebay-listing-export`, giving the final path `GET /api/external/ebay-listing-export`.
- Logger `ebayListingExportLogger` added to `configs/logger.config.ts`, per backend convention
  (every new module gets its own).

**Renamed same day, user's own catch:** originally built as "parts-price-export" (file names,
folder, logger, URL segment `/parts-price/awaiting-ebay-listing`) — user pointed out this is
misleading, since the endpoint never returns a price at all (that's the whole point — the external
system prices it, we just export the part details). Renamed everything to "ebay-listing-export"
(accurately describes the payload, not what the consumer later does with it) — service, controller,
route folder/file, logger, and the URL, which also got simplified from
`/parts-price/awaiting-ebay-listing` to just `/ebay-listing-export` (redundant sub-path dropped
since the parent segment now already says it; a future single-part endpoint would naturally become
`/ebay-listing-export/:taskPartId`). Confirmed zero leftover old-name references via grep before
calling it done. `tsc` clean.

**Final field list (user trimmed from the original 8-field proposal, then added `partNumber` back
2026-09-22):** `taskPartId`, `partName`, `sku`, `partNumber`, `originStatus`,
`vehicle: { vrm, make, model }`. No price/financial data, no staff names.

**2026-09-22 — coverage check + partNumber added.** Before adding, checked exactly how much
signal each identifier field actually carries, live, on the real "Awaiting eBay Listing" set (same
filter as the endpoint itself) — local AND production:
- Local: 1,532 total (matches the live Overview tile exactly — confirms the filter logic is
  correct). sku: 151 (10%). partNumber: 412 (27%). both: 113.
- Production: 1,581 total. sku: 210 (13%). partNumber: 502 (32%). both: 158.
`partNumber` has meaningfully better coverage than `sku` in both environments, which is why the
user asked to add it back in. `tsc` clean.

**Handoff process (walked the user through the real UI, verified via a subagent search of the
actual frontend code, not guessed):** Admin → "External API Clients" tile → create client → grant
exactly `GET:/api/external/parts-price/awaiting-ebay-listing` via the method-dropdown + path-input
composer → "Generate key" (shown once, copy-to-clipboard, mandatory
acknowledgement before the modal closes) → admin sends the key + URL to the external party
out-of-band. Confirmed the `API_CLIENT_MANAGEMENT_ACCESS` permission gate matches the backend's own
claim exactly (case-sensitive).

**2026-09-22 — single-part endpoint built too, plus a real staging debugging detour.**
User first floated a much bigger design (webhook notifications on status-change + a reverse
"send price back" endpoint) — gave a structured options/trade-offs analysis (pull vs push,
where a callback URL would live, what happens to a returned price) and did NOT implement, since it
was explicitly an "opinion request" with open questions, not a go-ahead. User then simplified:
just wanted the single-part GET endpoint after all (matching the ORIGINAL AG-299 scope's 2nd
requirement, deferred back on 2026-09-21). Built it:
- `getFlaggedPartByIdExport(taskPartId)` in the same service file — deliberately simpler scope
  than the bulk query: valid whenever `status` IN (Faulty/Incorrect/Not Required), no
  return-window/listing-state check at all (usable the instant a part is flagged, not only once it
  reaches "Awaiting eBay Listing"). Returns `null` for both "doesn't exist" and "not currently
  flagged" — controller maps both to a generic 404.
- Shared type renamed `TPartAwaitingEbayListing` → `TFlaggedPartExport` (used by both endpoints now).
- New route `GET /api/external/ebay-listing-export/:taskPartId`, added AFTER the `/` route in the
  same router file.
- **Operational gotcha flagged to user**: the existing client's grant (`GET:/api/external/ebay-listing-export`,
  no wildcard) does NOT cover the new `/:taskPartId` sub-path per the route-matcher's own
  collection-vs-sub-resource rule — they need to add a second grant,
  `GET:/api/external/ebay-listing-export/*`, on staging (and later prod) once deployed.
`tsc` clean both times.

**Staging debugging detour (same day, real production support, not part of AG-299's own scope but
worth recording since it blocks testing this ticket):** user got a "Not Found" testing the bulk
endpoint on staging. Diagnosed as a genuine backend 404 (matched `notFoundErrorHandler`'s exact
JSON shape) — not a proxy/DNS issue. Suggested testing the pre-existing `/api/external/health`
scaffold as a control; that request instead returned raw Next.js frontend HTML (build chunks, "Auto
Grid" title) — meaning that specific request hit the `carplanet` frontend, not the backend at all,
pointing to inconsistent staging routing/proxy behavior, not a code issue on this ticket. Mid-
investigation, user separately hit a `ER_NOT_SUPPORTED_AUTH_MODE` MySQL error — traced to the
backend's use of the legacy `mysql` npm package (`^2.16.0`, not `mysql2`), which doesn't support
MySQL 8's default `caching_sha2_password` auth plugin. Flagged as the likely root cause of the
whole staging mystery (new deploy's server process probably crash-looping at DB-connect on boot,
so whatever's actually answering requests on staging is a stale leftover process) — gave 2 fix
options (ALTER USER to `mysql_native_password`, or upgrade to `mysql2`) but did not apply either,
since it touches DB credentials/a real dependency change and needs the user's explicit choice.
**Unresolved as of last message** — waiting on user to confirm where exactly the MySQL error
occurred before deciding a fix.

**2026-09-22 (continued) — design pivoted twice more, landed on real push-webhook to a partner
endpoint.** After the single-part endpoint above, went through 3 iterations same day before landing
on the real shape:
1. Backend-only hook logging the snapshot internally on status-change — user said no visible effect,
   wanted a real frontend-visible call.
2. New internal JWT-protected `GET /task/part-status/:id/export-snapshot` + frontend `cpaxios.get`
   fired from `create_task_modal_new.tsx`'s `addUpdatePartStatus` right after the status PUT — built,
   `tsc` clean, but user then said this whole approach was wrong.
3. **Final, current design**: user provided the ACTUAL partner receiving endpoint —
   `POST http://172.31.5.203:8080/api/ebay-part-search`, request body = the exact same 6-field export
   shape already built. This is a genuine backend-to-backend push (172.31.5.203 is a private IP, not
   browser-reachable) — reverted both #1 and #2 entirely (frontend GET call removed, internal
   export-snapshot route/controller deleted) and replaced with:
   - New `services/inventory/ebay-part-search-webhook.service.ts` — `notifyEbayPartSearch(taskPartId)`,
     modeled directly on the existing `cardaddy-purchase-webhook.service.ts`'s `notifyVehiclePurchased`
     (found as an exact precedent for "fire external webhook, never block caller"): builds the
     snapshot via the already-existing `getFlaggedPartByIdExport`, POSTs to
     `process.env.EBAY_PART_SEARCH_API_URL` via axios (15s timeout), fully self-contained try/catch,
     never throws.
   - `task.service.ts`'s `updateTaskPartStatus` calls it **without `await`** (fire-and-forget from the
     caller, same convention as the CarDaddy precedent) every time status is (re-)set to
     Faulty/Incorrect/Not Required — confirmed with user this should fire EVERY time, not gated to
     first-flag-only.
   - New logger `ebayPartSearchWebhookLogger` added to `logger.config.ts`.
   - Confirmed with user: no auth header needed on this call (partner's own choice, matches the
     CarDaddy CRM webhook's own "unauthenticated by design" precedent).
   - `tsc` clean both repos.

**2026-09-23 — REPLACED the immediate-trigger design entirely with a daily scheduled queue.** User
gave a full 16-section written requirement (see conversation for verbatim text) demanding: no more
firing on status change; instead a daily job builds a queue of eligible parts (return window
expired, has sku/partNumber, no ebayPrice yet) and processes them ONE AT A TIME with 3-attempt
retry, never re-searching an already-priced part. I produced a full written analysis (current flow,
files affected, proposed architecture, DB impact, edge cases) BEFORE any code, per explicit
instruction — this is now a firmly established working pattern with this user for
architecture-level changes, not just this ticket (see [[opinion-request-is-not-a-go-ahead]] and
[[ask-before-applying-is-separate-step]] — same family of expectation).

**User caught a real design flaw in my first proposal** — I initially suggested a pure polling
"driver" job to move the whole queue forward. User pushed back: shouldn't the response itself
trigger the next part? They were right — corrected to a hybrid: the result-submission endpoint
advances the queue immediately/synchronously when a response arrives (no polling delay in the
normal case); the periodic job is ONLY a safety net for "partner never responds at all" (can't be
detected any other way, since absence of an event isn't itself an event — and has to be a
DB-timestamp check, not an in-memory `setTimeout`, since that wouldn't survive a restart). Worth
remembering: this user asks sharp, well-reasoned "why" questions on architecture proposals and
often improves them — treat pushback as genuine engineering review, not something to just
placate/reassert past.

**Confirmed parameters**: 30 min max wait per attempt, daily build at 2 AM London time, 5-min
safety-net interval (my recommendation, accepted), retry-eligibility never permanently capped —
re-queued on any future day for as long as still eligible (fresh 3-attempt budget each cycle, not a
lifetime cap).

**Found excellent existing precedent** for almost this entire architecture already in the codebase:
`services/invoice/px-stock-recovery.service.ts` + `entities/invoice/px-stock-recovery-log.entity.ts`
— a "scan for candidates, track status/attemptCount/lastError in a dedicated log table, save after
every attempt" cron pattern, and `configs/cron.config.ts`'s own convention (env-gated CronJob
instances, London timezone, single `APP_WITH_CRON`-tagged cluster worker — confirmed via
`worker.ts`/`index.ts` — which for free already prevents duplicate scheduler execution across
processes, no new locking needed).

**Built**: new `ebay_price_queue` table/entity (modeled on, NOT editing, `px_stock_recovery_log`),
new `ebay-price-queue.service.ts` (`buildDailyQueue`, `sendNextIfIdle`, `checkStalledEntries`,
`markResultAndAdvance`), extended the shared "Awaiting eBay Listing" WHERE-clause (refactored into
`applyAwaitingEbayListingFilters()` in `ebay-listing-export.service.ts`, reused by both the bulk
export API and the new queue-eligibility query — never duplicated), removed the immediate-trigger
call from `task.service.ts`'s `updateTaskPartStatus` entirely, 2 new cron jobs registered following
the file's exact existing conventions. `ebay-part-search-webhook.service.ts` (built the round
before) reused completely unchanged — just called from a new place. Zero frontend changes needed.
2 migrations created (this one + the earlier `ebay_price`/`ebay_result_raw`/`ebay_searched_date`
one), neither run. Backend `tsc` clean.

**2026-09-23 (continued) — real local testing found and fixed a genuine bug.** User ran migrations
themselves, then tested locally with 3 temporary values (cron time moved to whatever "now+a few min"
was, `.slice(0,5)` cap on the daily queue build, and `cronConfig.start()` temporarily uncommented in
`index-dev-1.ts` since local dev never runs cron by default). First test picked up `taskPartId=30` —
a part with `status="Used"` — which should have been impossible. Root cause: a genuine SQL
operator-precedence bug in `getEbayPriceQueueEligiblePartIds()` — the sku-or-partNumber `OR`
condition wasn't wrapped in an outer set of parens, so it silently broke the ENTIRE WHERE clause
(`... AND (sku present) OR (partNumber present AND ebayPrice IS NULL)` instead of `... AND (sku
present OR partNumber present)`) — verified directly against the local DB: the buggy query matched
**14,932** rows (nearly the whole table); the fixed query matches the correct **466**. Fixed by
adding one more set of outer parens around the whole OR expression. This is exactly the kind of bug
that `tsc`/lint can never catch (valid TypeScript, valid SQL, just semantically wrong) — only running
it for real against real data surfaced it. **Lesson: raw-SQL boolean conditions built via multiple
chained `.andWhere()` calls need every OR-containing clause independently wrapped in its own full
outer parens, checked by literally reading the generated WHERE clause's boolean logic, not just
confirming the query runs without error.**

Also discovered a real side-effect risk from the "just uncomment `cronConfig.start()`" testing
shortcut: it enables EVERY cron job in the file locally, not just the eBay one, and at least one of
them (`scanAndRecoverMissingPXStock`) is not read-only — it created 2 real vehicle stock records
(ids 31190/31189) on the user's local DB as a side effect of testing something unrelated. Cleaned up
the queue table contamination from the buggy-query test run (5 bad rows deleted) but the 2 created
PX stock vehicles were left as-is (not undone) — noted here in case that surfaces confusion later.
All 3 temporary testing values reverted afterward (cron time back to `"0 2 * * *"`, `.slice(0,5)`
removed, `cronConfig.start()` re-commented + its now-unused import removed) — confirmed via
`git diff` that `index-dev-1.ts` is byte-identical to its last committed state.

**2026-09-23 (continued) — "Estimated eBay Recoverable Price" = ebayPrice × 0.90, wired into both
UI surfaces.** User spotted a real inconsistency: the Overview tile's "Awaiting eBay Listing"
total (`£X (Estimated Recoverable Value)`) was STILL using only the old AG-270 AI-estimate bracket
formula for every part, with zero awareness of the new `ebayPrice` column — confirmed by reading
`inventory.service.ts`'s `getPartsResolutionBuckets` raw SQL directly. Then refined the requirement:
rather than showing the raw `ebayPrice` anywhere, both surfaces should show a suggested LISTING
price — 10% below the cheapest matching eBay listing found (`ebayPrice × 0.90`), to undercut the
market rather than just repeat it back. `ebayPrice` itself stays stored as the raw true figure — the
10%-off is applied only at display/aggregation time, not baked into what's persisted.
- `task_parts_status_badge.tsx` — new `estimatedEbayRecoverablePrice = ebayPrice * 0.9`; tooltip
  line relabeled from "eBay Price" to **"Estimated eBay Recoverable Price"**, still replacing (not
  supplementing) the AI-estimate fallback line exactly as before.
- `inventory.service.ts`'s `getPartsResolutionBuckets` — `readyForSaleRecoverableValue`'s SQL now
  checks `ebayPrice IS NOT NULL` first (→ `ebayPrice * 0.90`), falling through to the unchanged
  AG-270 bracket formula only when there's no `ebayPrice`. Comment explicitly flags this must be
  kept in sync with the frontend's copy of the same 0.9 multiplier (manual sync, same pre-existing
  convention as the bracket formula itself already has, since one runs in SQL and the other in JS).
- No schema/migration needed — just multiplying an already-available number in 2 places.
- Both repos `tsc --noEmit` clean.

**2026-09-23 (continued) — same "Estimated eBay Recoverable Price" concept extended to a THIRD
surface: Part Returns Audit's "Mark as Listed" popup.** User wants the same on-hover suggested price
(ebayPrice × 0.90, else AI-estimate fallback) shown on an info icon next to the "Asking price (£)"
field when marking a part as Listed — same labels, same computation, so the person filling in the
asking price has the suggestion right there.

Traced the actual data flow: Part Returns Audit table → `POST /inventory/task-parts-non-conforming-section`
→ `inventory.service.ts`'s `taskPartsNonConformingSection` → `NonConformingSignOffModal` →
`PartNotePanel` (which actually owns the Listed/Resold/Refitted/Scrapped action fields, despite its
name suggesting it's just the comment thread). Neither `invoiceUnitPrice` nor `ebayPrice` were in
this query's SELECT list or the frontend `NonConformingPart` type at all — added both.
- `inventory.service.ts` — added `taskPart.invoiceUnitPrice`/`taskPart.ebayPrice` to
  `taskPartsNonConformingSection`'s SELECT list.
- `types/inventory.ts` — `NonConformingPart` gains `invoiceUnitPrice`/`ebayPrice`.
- `non_conforming_sign_off_modal.tsx` — passes `poUnitPrice`/`invoiceUnitPrice`/`ebayPrice`/
  `partArrivedDate` down to `PartNotePanel` (which already had `Tooltip`/`InfoOutlinedIcon`
  available at the modal level, but the actual Asking-price field lives one level down).
- `part_note_panel.tsx` — new props, computes `estimatedRecoverablePrice` itself (same priority
  rule: `ebayPrice != null` → `ebayPrice * 0.9`, else `computeSuggestedResalePrice(...)` imported
  from `non_conforming_tab.tsx`, same function the Overview/Task-Card surfaces already use — no
  duplicated formula). Info icon + `Tooltip` added inline next to the Asking price input, same
  "Estimated eBay Recoverable Price" / "Recoverable Price (AI estimated)" label wording reused
  verbatim from the Task Card tooltip for consistency across all 3 surfaces now.
- Both repos `tsc --noEmit` clean.

**2026-09-24 — excluded single-digit sku/partNumber junk values from the queue eligibility check.**
User found 8 real local-DB parts with `part_number = "0"` or `"1"` (placeholder/junk data, no real
identifier) that were incorrectly qualifying for the cron. Fixed in
`getEbayPriceQueueEligiblePartIds()` only (NOT the shared `applyAwaitingEbayListingFilters` — that's
also used by the bulk export/Overview tile, which shouldn't be affected by this cron-specific rule):
strengthened the sku/partNumber-presence check with `NOT REGEXP '^[0-9]$'` (matches only a
value that's exactly one character AND a digit — "10"/"A1"/"0000123" etc. unaffected). Initially
scoped to partNumber only per the user's first phrasing; user confirmed the SAME rule applies to
`sku` too — a part is excluded from the queue only when BOTH sku and partNumber are
missing/empty/single-digit (needs just one usable identifier via either field to qualify). Verified
directly against local DB before AND after the fix (8 junk-only parts found, correctly excluded;
regex sanity-checked against "10"/"1"/"A1"). `tsc` clean.

**Also same-day: staging cron gate toggled on/off/on again during interactive testing.**
`EBAY_PRICE_QUEUE_ELIGIBLE_ENVS` in `cron.config.ts` — user repeatedly asked to pause and resume
staging testing (remove/re-add `"staging"` from this array) and to change the eBay build job's TEMP
testing time (12:01 PM → 12:20 PM → 1:15 PM → 1:20 PM → 2:00 PM → 3:00 PM IST, each precisely
converted to UK/BST time via Node's Intl, not hand math). Current state as of this writing:
`"staging"` IS included (testing resumed), build job TEMP time is 3:00 PM IST (`"30 10 * * *"`).
**Both are temporary — check `tasks/todo.md`'s live tracking note for the actual current values
before deploying anywhere**, this memory file's own snapshot will go stale fast given how often
these were toggled. Confirmed staging's `NODE_ENV=staging` (not in the original
`ELIGIBLE_ENV_TO_RUN_CRONS` list) is exactly why a dedicated gate was needed for these 2 jobs in the
first place — added earlier the same day (see the two-layer-gate explanation above in this file if
resuming this thread later).

**Still open / next step (as of 2026-09-23, current)**: user has run both migrations locally and
confirmed the queue works end-to-end for real (taskPartId=313 "Fuel Flap Lock" successfully got
`ebayPrice=55.33` saved via the actual partner round-trip during testing). Local testing is
effectively DONE and successful. Not yet done: staging/production still need the same 2 migrations
run + `EBAY_PART_SEARCH_API_URL` set + the `POST:/api/external/ebay-listing-export/*` grant added on
each environment's own API client (all environment-specific setup, not code). Also still open: the
partner's actual submitted-result payload shape doesn't match our documented contract — they send
the result FLAT (no `{ "ebayResult": ... }` wrapper), and our endpoint currently only reads
`req.body.ebayResult` — flagged to user with 2 fix options (ask partner to add the wrapper, or make
our endpoint tolerant of both shapes), **not yet resolved/chosen** as of this writing. The 300000ms
timeout mystery from 2026-09-22 (below) was never resolved but is now moot — the new queue design
doesn't depend on that response body at all.

**2026-09-22 (historical) — the 300000ms timeout mystery, and the prior architecture pivot
to two independent calls.** Spent a long debugging arc chasing an inexplicable
"timeout of 300000ms exceeded" axios error (user kept testing against 3 different local-network IPs
— 172.31.5.203, 192.168.0.86, 192.168.1.27 — for the partner's receiving server). Proved via direct
axios source inspection that this exact error text can ONLY come from axios's own `config.timeout`
being truthy — yet grepped the entire backend and found NO code setting `300000` anywhere (not
`axios.defaults`, no interceptors, only one axios install, no tsconfig path alias). Went further and
compared the failing request's UTC log timestamp against the actual OS process-start time
(`Get-CimInstance Win32_Process`) and proved a request that failed with "300000ms" happened ~7.5 min
into a process that started AFTER the code was already edited to have NO timeout at all — ruling out
"stale process/nodemon didn't reload" as the explanation. **Root cause never fully resolved** — user
pivoted away from chasing it once a better architecture made the whole question moot (see below).

**Final architecture (implemented, this is the one that's actually built):** split into two fully
independent calls instead of one synchronous request/response:
1. **Trigger** (`ebay-part-search-webhook.service.ts`'s `notifyEbayPartSearch`, unchanged in
   structure) — POST to `EBAY_PART_SEARCH_API_URL`, fire-and-forget, no timeout. Simplified to ONLY
   log whether the trigger was accepted — no longer parses/logs the response body as if it carried
   the answer, since that's what caused the whole synchronous-wait problem.
2. **Result submission** (NEW) — `POST /api/external/ebay-listing-export/:taskPartId/results`, same
   API-key auth as the rest of AG-299, body `{ "ebayResult": <anything> }` — deliberately NOT
   strictly schema-validated since the partner's format isn't a fixed contract (user's explicit
   requirement: "the other user can send any thing"). The partner calls this whenever their search
   actually finishes, independent of the trigger — could be seconds or hours later.

**Pricing rule (user's explicit business logic, not something I designed):**
- Part status **Faulty** → only listings with `type === "Parts only"` count; take the MINIMUM price
  among those as `ebayPrice`. No average.
- Part status **Not Required** or **Incorrect** → only listings with `type === "Brand New"` count;
  take the MINIMUM price among those.
- No matching-type listing (or no results at all) → `ebayPrice` stays `null`; the UI tooltip already
  falls back to the existing AI-estimated "Recoverable Price" (AG-270's `computeSuggestedResalePrice`)
  automatically — no new fallback logic needed, that formula already existed.

**Built (this round):**
- Migration `1790088733577-add-ebay-search-result-to-task-part.ts` (3 columns on `task_part`:
  `ebay_price` decimal nullable, `ebay_result_raw` JSON nullable, `ebay_searched_date` datetime
  nullable) — **NOT run**, per standing migration rule.
- `task-part.entity.ts` — `ebayPrice`, `ebayResultRaw`, `ebaySearchedDate` columns added.
- `ebay-listing-export.service.ts` — new `submitEbaySearchResult(taskPartId, ebayResult)`: looks up
  the part fresh (uses whatever its CURRENT status is at submission time to pick Faulty→"parts only"
  vs NotRequired/Incorrect→"brand new" — noted as a known edge case if status changes again between
  trigger and result), defensively extracts an array from the loose `ebayResult` payload (handles it
  being the array directly, or nested under `.ebayResults`), parses price strings (strips currency
  symbols/commas), filters by required type, takes the min, saves `ebayPrice` (or null) +
  `ebayResultRaw` (always, raw, whatever shape) + `ebaySearchedDate` (always, now) regardless of
  whether a price was computed. Returns null only if the part doesn't exist (404).
- `ebay-listing-export.controller.ts` / `.route.ts` — new `POST /:taskPartId/results` handler +
  route, same file/pattern as the other 2 AG-299 endpoints.
- Frontend: `types/task.ts` (`TaskPart.ebayPrice`), `task_parts_status_badge.tsx` (`ebayPrice` prop,
  `hasEbayPrice` derived value, tooltip line — shows "eBay Price: $X" INSTEAD OF "Recoverable Price
  (AI estimated)" when available, never both), `create_task_modal_new.tsx` (passes `part.ebayPrice`
  through). No frontend query/select changes needed — `getTasks` uses a plain `TaskPartRepository.find()`,
  so new entity columns flow through automatically.
- Both repos `tsc --noEmit` clean.

**Not yet done:** migration not run (user's call, per standing rule); the new POST route grant
(`POST:/api/external/ebay-listing-export/*`) needs adding to the existing API client once deployed —
same operational note as the single-part GET endpoint before it.

**Requirement, as given:**
1. Bulk endpoint — returns every part currently in "Awaiting eBay Listing" (the existing
   `RESOLUTION_STAGE: READY_FOR_SALE` bucket), so an external server (on another machine) can pull
   them all and run its own price extraction/estimation.
2. Single-part endpoint — returns one part's details, for parts whose status is Faulty, Incorrect,
   or Not Required.

**Interaction model — confirmed with user:** both are **pull** (external server calls in), not a
push/webhook fired the moment a part's status changes. Flagged as an explicit assumption before
building; user confirmed by proceeding to ticket creation without correcting it.

**Auth design (proposed, not yet built):** no logged-in AutoGrid user in this flow, so the normal
JWT `authenticate` middleware doesn't apply. Following the existing `/publicurl/webhooks/etc-stock-transfer`
precedent (its own `ETC_TRANSFER_SECRET`) — a **new**, dedicated secret (not reusing ETC's, so
rotating one never breaks the other), checked via a request header, routes under `/publicurl`.

**Proposed endpoint shape (not yet confirmed against real payload needs):**
- `GET /publicurl/parts-price/awaiting-ebay-listing` — bulk, reusing the exact same query logic the
  drilldown already uses for `READY_FOR_SALE` (avoids drift from what the UI shows).
- `GET /publicurl/parts-price/:taskPartId` — single, 404/403 unless status is Faulty/Incorrect/Not
  Required (don't leak arbitrary task parts to an external caller).
- Proposed default payload per part: part name, origin status, supplier, price actually paid
  (invoice price falling back to PO price), days since arrival, vehicle make/model. Not yet
  confirmed with user — may need adjusting once the external consumer's real needs are known.

**No user-permission gate** — access control is the shared secret itself, not a Team Permissions
checkbox (there's no logged-in user in this flow to gate).

**Ticket-hierarchy note:** originally asked to create this as a sub-task of AG-270 — Jira rejected it
(AG-270 is itself a Sub-task; Jira doesn't allow sub-tasks of sub-tasks). Redirected to AG-260
instead (a Story, so it can hold sub-tasks) — and on reflection AG-260 is arguably the *better* fit
anyway, since AG-260's own description is what defined the "Ready for sale"/"Awaiting eBay Listing"
bucket and the `TaskPart` listing-metadata model this API actually exports; AG-270 is about
resale-price *methodology*, this ticket is about exporting the *data* that methodology would consume.

**Related:** [[issue-AG-270-ai-resale-price-research]] (the research this API supports — external
price extraction instead of an internal Gemini call), [[issue-AG-271-ai-price-estimation-research]]
(same research family, different missing-price problem — worth reusing that ticket's calibration
lesson if an internal AI approach is ever tried here too).

---

## HISTORY

- 2026-09-21: User gave the new requirement directly in chat (no ticket yet). Explained the plan
  (2 pull endpoints, ETC-webhook-style shared-secret auth, proposed payload), flagged the
  push-vs-pull ambiguity explicitly per [[prefers-conversational]] (asked inline, not via a
  pick-list), and asked for a ticket number before implementing. User said "create one under AG-270
  as a subtask" — Jira rejected (sub-task of a sub-task not allowed) — redirected to AG-260 on the
  user's own follow-up ask for my opinion; created as AG-299. Memory file created immediately once
  the real ticket number existed, per [[create-ticket-file-immediately-on-open]]. Implementation not
  yet started — next step is to actually build it, still pending explicit go-ahead (the original
  explanation message already asked permission; that permission request predates the ticket-number
  detour and should be treated as still open, not yet re-confirmed after the AG-299 detour).
