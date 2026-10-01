---
name: issue-AUT-3639-booking-link-multiple-vehicles
---

## NOW

- **Status (2026-10-02): batch 3 coded, UNCOMMITTED, tsc clean, not browser-tested.**
  Batch 2 appears committed (only batch-3 files dirty). Batch 3:
  - Route bug: `PUT /booking/swap-primary-vehicle` was below `PUT /booking/:id` →
    `"id" must be a number`; moved above it (comment says keep it there).
  - Decision (user): keep 2 endpoints. `update-booking-vehicle` = fix wrong car
    (old drops off); Make Primary = swap (old stays extra). Both write only
    `booking.vehicle_id` + `updatedAt`.
  - Deferred item 1 DONE: `updateBookingVehicle` rejects `isMultipleVehicle`
    bookings; car icon greyed (Booking Details + Sales Diary list); reg-mismatch
    "Switch" replaced by a Make Primary note on those rows.
  - `swapPrimaryVehicle`: reads in tx with booking pessimistic_write lock; stale
    is_primary=1 rows for non-primary vehicles → REMOVED; returns
    `extraVehicles`; frontend `router.replace` with new `vid`.
  - New open Q18: `advertPriceAmount` / lead vehicle not updated on swap.
  - Next: user tests + commits; open Q's 1–13, 15–18.
- **Status (2026-10-01): batch 1 committed (be f126fe859 / fe f485c6238, branch
  AKASH/Feature-AG-312-adding-extravehicle); batch 2 (price hold/cap/counts/tx)
  UNCOMMITTED. Migration HAS been run locally (confirmed via read-only local DB
  check 01-10). Not yet tested.** Deferred items 2 (transaction), 5 (cap) and the
  pricing-hold/stored-count parts of 7 are DONE — the list below is the 29-09
  snapshot. New finding 01-10: Pricing Tool price hold is display-only — Update
  Price uses stored `has_suggested_price_checked` with no viewing re-check
  (questions.md #17).
- **Status (2026-09-29): IMPLEMENTATION IN PROGRESS — first pass coded, not yet
  tested, migration NOT run.** Scope: schema + backend add/remove/swap + Booking
  Details page UI only. Full file list + deferred items in `tasks/todo.md` at
  projects root.
  - New table `booking_linked_vehicle` (id, booking_id, vehicle_id, is_primary,
    status ACTIVE/REMOVED, removed_at, removed_by, created_by, createdAt,
    updatedAt) — one row per vehicle per booking, plain int cols, NO
    FKs/relations. created_by/removed_by set server-side from `req.user.id` in
    the controllers (never from request body). Migration
    `1790683453257-create-booking-linked-vehicle-table.ts` also adds
    `booking.is_multiple_vehicle`.
  - Primary row only appears in the table after its first swap-demotion;
    `booking.vehicle_id` stays source of truth for primary.
  - Endpoints: POST/DELETE `/sales-diary/booking/extra-vehicle`, PUT
    `/sales-diary/booking/swap-primary-vehicle`. `bookingsForVehicle` now
    returns `extraVehicles` + `isMultipleVehicle`.
  - Frontend: `booking_vehicle_details.tsx` (extras list, +, Make Primary,
    remove), `select_vehicle_modal.tsx` (`mode="addExtra"`),
    `sales_diary_list.tsx` VRM cell "+X" pill + AGTooltip hover list (layout
    (b): pill right after reg, following icons offset 26px only on rows with
    extras, column 145→170px). Backend for it: `booking.isMultipleVehicle` in
    `SALES_DIARY_LIST_COLUMNS` (booking.utils.ts) + shared helper
    `getExtraVehiclesByBookingId` used by `filteredBookingsNew` (SALES_DIARY
    type only) and `bookingsForVehicle`.
  - **Batch 2 done (2026-09-30), client answers applied:** price hold extended
    to active extras + deleted bookings no longer hold price (primary too —
    user decision); cap 3 extras (4 total) backend guard + "+" disabled at 3;
    add/remove wrapped in transaction with booking row pessimistic lock
    (deferred item 2 closed); cron `updateBookingsCountForVehicles` UNION so
    extras count in all 3 stored counts (upcoming/previous/total — user
    decision). No migration. Deferred items still open: 1 (old swap icon not
    table-aware), 3 (DELETE with body), 4 (permissions), 6, 7 (rest).
  - **Four living docs, update ALL with every change** (see
    [[aut3639-update-four-docs-every-change]]): feature doc, backend change log,
    `date-wise-daily-update.md` (plain-language daily points), `questions.md`
    (16 open / 9 resolved as of 30-09-26).
  - Jira: {AG-312: Add, remove and swap extra vehicles on a booking} — Sub-task
    under AG-311, assigned Jitendra. Description in /jira-ticket format (no
    code terms); technical notes posted as comment 14887. Use /jira-ticket
    format for any further AG tickets on this work.
  - Backend change log for this ticket (tables/columns/endpoints, newest
    first): `my-docs/projects/Booking-AUT-3639-5th-project/project-log-whatimptablesormigration-added.md`
    — append an entry every time a table/column/backend feature changes.
  - **DEFERRED — required future changes (user explicitly parked these,
    2026-09-29; do NOT treat as done):**
    1. **Old swap icon not table-aware (real data-consistency bug).**
       `updateBookingVehicle` (`sales-diary.service.ts`, the existing
       `PUT /sales-diary/update-booking-vehicle`, fired by the vehicle-change
       icon in `booking_vehicle_details.tsx` via `select_vehicle_modal.tsx`
       mode="swap") only sets `booking.vehicle` — it never touches
       `booking_linked_vehicle`. If staff use it to switch primary to a vehicle
       that's already an ACTIVE extra → that vehicle is both primary
       (booking.vehicle_id) and an extra row (is_primary=0). The old primary
       also vanishes instead of becoming an extra. Fix options: make
       updateBookingVehicle table-aware (route through swapPrimaryVehicle
       logic when the target is an extra / demote old primary into the table),
       or hide/redirect the old icon when `isMultipleVehicle` is true.
    2. **add/remove not transactional.** `addExtraVehicle` and
       `removeExtraVehicle` do the booking_linked_vehicle save and the
       `booking.is_multiple_vehicle` save as two separate writes → a crash
       between them leaves the flag out of sync. Fix: wrap each in
       `AppDataSource.transaction(async (manager) => …)` like
       `swapPrimaryVehicle` already is (pattern used in 11 other services,
       e.g. task.service.ts:1649).
    3. **DELETE with JSON body.** `DELETE /sales-diary/booking/extra-vehicle`
       sends bookingId/vehicleId in the body (frontend: `cpaxios.delete(url,
       { data })` in `booking_vehicle_details.tsx`). Some proxies/LBs strip
       DELETE bodies → removal could fail silently on staging. Fix: switch to
       PUT/POST, or move ids into the URL path.
    4. No permission gate on add/remove/swap (matches existing swap icon,
       which has none either; no BOOKING_VEHICLE-style permission exists).
    5. No cap on extras (prod data: customers never view >3 total).
    (Living feature doc — single source of truth for design/schema/APIs/files/
    deferred items/test checklist:
    `my-docs/projects/Booking-AUT-3639-5th-project/BOOKING_LINKING_MULTIPLT_VEHICLE_WITH_SINGLE_BOOKING_3639.md`.
    Update it whenever this ticket's state changes.)
    6. Primary vehicle only gets a booking_linked_vehicle row after its first
       swap-demotion — `createBooking` doesn't insert one. If every booking's
       primary must always be in the table, needs a createBooking insert +
       backfill of existing bookings.
    7. Out-of-scope consumers still single-vehicle: Sales Diary "+X"
       indicator/reg search, Vehicle/Stock page (bookingsForVehicle only
       matches primary via `booking.vehicle_id`), pricing-tool 7-day hold
       (`getUpcomingBookingCount`/`List`), Internal Transport, Live ETA,
       reporting, WhatsApp/Audiofy/email/DocuSign content — full list in
       `my-docs/projects/Booking-AUT-3639-5th-project/AUT-3639_Requirement_Impact_Analysis.md`.

- **Status (2026-09-25): Full requirement & impact analysis DONE (backend + frontend + complete
  blast-radius sweep).** No code touched — analysis only, per explicit instruction to ask
  permission before implementing. Delivered as
  `my-docs/projects/Booking-AUT-3639-5th-project/AUT-3639_Requirement_Impact_Analysis.md` +
  matching `.docx`, both rewritten as one integrated document (not patched addenda) after the user
  correctly pushed back that the first pass only covered the ticket's own named modules.
- **Ticket:** carplanetuk.atlassian.net, "Feature Time Estimation" status, priority Medium,
  assignee Akash Robert, reporter Hemant Devde, parent epic {AUT-833: Admin}. Ask: let one booking
  reference multiple vehicles via an additive `booking_vehicle` child table (client's own
  recommendation); existing `vehicle_id` on `Booking` keeps meaning "primary vehicle"; one
  appointment/visit/sale count per booking regardless of vehicle count.
- **Core design (proposed, pending sign-off):** new `booking_vehicle` table (booking_id FK,
  vehicle_id FK, audit cols); primary-swap = insert-extra-row + reassign `booking.vehicle_id` +
  remove-the-promoted-row transaction. Follows the Task/TaskPart child-table precedent; a second,
  more domain-relevant precedent exists too — `booking_reservation_log` (dedicated booking↔vehicle
  pairing table for reservation locking, migration `1783200000000-create-booking-reservation-log.ts`).
- **Pricing-tool "~7 day hold" — fully located** (was the one open unknown through 3 passes,
  found on the 4th): `sales-diary.service.ts::getUpcomingBookingCount`/`getUpcomingBookingCountList`
  (~L2674/L2711) query `category=VIEWING` bookings in the next 7 days, filtered on a single
  `vehicle_id` — feeds `pricing-tools.service.ts`'s `hasAbleToUpdatePrice` flag on `Vehicle`. No
  stored lock/timestamp — recomputed live every request. Fix is precisely scoped: extend those two
  functions to also check `booking_vehicle`. No separate primary-swap recalculation needed since
  it's live-computed.
- **Real footprint is ~3x the ticket's own 7 named modules.** Full grep-driven sweep (both repos,
  not ticket-scoped) found ~14 more backend files and ~6 more frontend files reading
  `booking.vehicle`/`vehicleId` directly: customer-facing DocuSign PDFs
  (`docusign-html-templates.ts`), two 3rd-party integrations (Audiofy — `booking-audiofy.service.ts`;
  Wildjar call-tracking webhook — `wildjar.service.ts`), SMS (`viewing-confirmation.service.ts`) +
  email (`sendEmailToCustomer`), Customer 360, External Deliveries, 3 extra reporting services
  beyond the known one (`manager-report.service.ts`, `reports.service.ts`), Prep Center, Retention
  module, the Vehicle service itself, Vehicle Movement Planner, FTC-Pro (Sales Diary sub-module);
  frontend: `booking_schedule_modal.tsx`, `customer-360/visit_summary_table.tsx`,
  `external-deliveries.tsx`, DNA/FTC sub-lists. DB: `audit_log` and `i_love_pdf_logs` also pair
  `booking_id`+`vehicle_id`.
- **Permissions:** confirmed (not assumed) that no `BOOKING_VEHICLE`-style permission exists
  anywhere — must be built from scratch regardless of how Step 8's role-gating question is answered.
- **Second document added:** `AUT-3639_Existing_Flow_And_MultiVehicle_Impact_Analysis.md` +
  `.docx`, same folder — a companion doc tracing the **as-is execution flow** (not just which
  files touch `booking.vehicle`, but *when/how* each fires relative to a booking save). Traced
  via a 5th subagent with exact line numbers. Key output: a 4-tier risk classification —
  **Tier 1** sync-in-request (`booking_reservation_log` L626/L3776, `audit_log` L799/L4838) —
  highest risk, executes inside the booking-save transaction; **Tier 2** async fire-and-forget
  (WhatsApp confirmation via SuperchatAPI — corrected from "SMS" in the first doc —, Live ETA
  refresh, Audiofy on check-in only) — silent-staleness risk; **Tier 3** separate later user
  actions (email, DocuSign/test-drive PDF, Start Deal/VehicleDeal) — product-scope question, not
  urgent; **Tier 4** pure on-demand reads (Pricing Tool, Sales Diary, reporting, etc.) — lowest
  risk, just query fixes. Also confirmed Wildjar webhook doesn't actually read `Booking.vehicle`
  at all (joins a different relation) — excluded from scope. Surfaced an existing gap:
  `updateBookingVehicle` never calls `createReservationLog`, even for reservation bookings —
  pre-existing inconsistency, not introduced by this ticket, needs a decision either way.
- **Status (2026-09-25, final): Reconciled — doc 1 updated with doc 2's findings.** User asked
  directly whether the flow trace actually changed the main impact-analysis doc, or was just extra
  detail. Answer: it did change it, in 3 concrete ways, now folded into
  `AUT-3639_Requirement_Impact_Analysis.md`/`.docx`:
  1. **Factual correction** — Step 2.4 table had `viewing-confirmation.service.ts` labeled
     "SMS"; corrected to WhatsApp via SuperchatAPI everywhere it appears (2.4 table, Step 8 Q14,
     "Where I'd Push Back").
  2. **Corrected a wrong claim** — Step 5's Sequence section previously said all downstream
     consumers "pick this up on next read, no separate propagation needed" — **false** for Tier
     1/2 (reservation log, audit log, WhatsApp, Live ETA refresh, Audiofy): these are one-shot
     calls at specific code points in `createBooking`/`updateBooking`/`updateBookingVehicle` and
     will NOT automatically cover extras without an explicit code change at each call site. Fixed.
  3. **Added a new Step 2.5a "Trigger-tier reframing"** section (Tier 1-4, same framework as doc
     2) that now drives Step 3/4/6's prioritization — Step 6's flat "12. [12 files], scoping
     decision needed" bucket was split into 12a (Tier 1, ship with core, no deferral), 12b (Tier 2,
     ship with core pending content sign-off), 12c (Tier 3, explicitly deferred), 12d (Wildjar,
     confirmed excluded), 12e (presumed-but-not-trace-confirmed Tier 4 — explicitly flagged as a
     confidence-level distinction, not overclaimed as verified).
  - Step 8 consolidated into ONE complete list (11 original + 6 new flow-derived questions = 17
    total) so this document is self-contained rather than requiring cross-reference to doc 2.
  - Step 7 edge cases gained 5 new rows from the flow trace (reservation-log gap, audit scope,
    Live ETA staleness, Audiofy payload format, deal-swap non-retroactivity).
- **Status (2026-09-28): Internal AG twin created.** New epic {AG-310: Admin} (mirrors client's
  {AUT-833: Admin}) and Story {AG-311: Booking — link multiple vehicles to a single booking
  (AUT-3639)}, parent-attachment verified, unassigned per instruction. Both analysis `.docx` files
  attached to AG-311 (confirmed via re-fetch: 18349 + 15951 bytes, exact match). Remote link added
  AG-311 → `carplanetuk.atlassian.net/browse/AUT-3639` (verified via GET; no reverse link added on
  the client's board — that direction wasn't asked for and stays opt-in per the aut-jira-access
  skill's caution about exposing AG numbers to the client). Issue type "Story" chosen after
  checking prior AG tickets' actual types (AG-213 = Story for brand-new functionality under an
  epic; AG-166/189/234 = Change Request for tweaks to existing features; AG-236/237 = Sub-task) —
  matches this ticket being wholly new functionality. No multiple sub-tickets created yet per
  explicit instruction — analysis still in progress, will split later.
- **Note:** the `INTERNAL_JIRA_API_TOKEN` credential in `.jira-bridge.env` is registered to Akash
  Robert's account, not Jitendra's — attachments on AG-311 show his name as uploader. Not an error,
  just how that shared script credential is configured.
- **Next action:** none — both documents fully reconciled/self-contained, AG twin created and
  verified. Two highest-leverage open items unchanged: (1) scoping decision on which of the ~20
  newly-surfaced consumer files are in-scope vs. fast-follow (Step 8 Q9), (2) per-tier product
  content decisions (WhatsApp/Audiofy/email — several need sign-off from whoever owns those
  integrations). Awaiting user/stakeholder direction before any implementation or ticket-splitting.
- **Standing lesson from this ticket:** [[map-to-existing-system-means-exhaustive-sweep]] — saved
  as a global feedback rule so future requirement-analysis prompts run the exhaustive sweep at
  Step 2 by default, not as a follow-up requested after the fact.

---

## HISTORY

- 2026-09-25 — Fetched AUT-3639 from client Jira via `aut-jira-access` skill (REST call through
  PowerShell since no Python interpreter is available on this machine; env creds found at
  `C:\Users\jiten\.jira-bridge.env`, not the `C:\Users\akash\...` path the skill doc names).
  Registered as new active ticket per user instruction.
- 2026-09-25 — First analysis pass: backend (Booking entity/controller/service, related modules,
  Task/TaskPart child-table precedent) + frontend (Booking Screen, Sales Diary, Vehicle/Stock,
  Internal Transport, Live ETA, Reporting, SalesDiaryContext/VehicleContext) mapped via two Explore
  subagents. Delivered as `.md` + `.docx` in `Booking-AUT-3639-5th-project/` (local spec doc there,
  `BOOKING_LINKING_MULTIPLT_VEHICLE_WITH_SINGLE_BOOKING_3639.md`, was empty — no existing spec to
  reconcile against). Pricing-tool "~7 day hold" logic not located in this pass. User deleted an
  unformatted scratch file (`initial-first-analysis-fromtheticket.md`) after the proper pair was
  created.
- 2026-09-25 — User pushed back: analysis only covered the ticket's own named modules, asked to
  find "each and every component" affected. Ran a second, code-driven (grep-first, not
  ticket-first) sweep of both repos for every consumer of `booking.vehicle`/`vehicleId`. Found ~19
  additional files (DocuSign, Audiofy, SMS, Customer 360, External Deliveries, 2 more reporting
  services, Prep Center, Retention, Vehicle service, Vehicle Movement Planner, FTC-Pro,
  `booking_reservation_log` table, `audit_log`, `i_love_pdf_logs`, no `BOOKING_VEHICLE` permission
  found). Patched into both docs as a "Step 2b" addendum section.
- 2026-09-25 — User asked directly why the exhaustive sweep wasn't done on the first pass, given
  they'd provided a detailed 8-step structured prompt that already asked to "map to existing
  system." Correct critique — the prompt was fine, execution narrowed Step 2 to the ticket's own
  vocabulary instead of treating it as an independent, ticket-agnostic step. Saved
  [[map-to-existing-system-means-exhaustive-sweep]] as a standing global feedback rule.
- 2026-09-25 — User asked to redo the whole analysis and find everything related. Ran one more
  targeted search specifically for the still-missing pricing-hold mechanism (found — see NOW) and
  a final consumer sweep (found Wildjar webhook + `sendEmailToCustomer`, excluded `docusign.service.ts`
  and `webhook/call-log.service.ts` as bookingId-only). Fully rewrote both `.md` and `.docx` as one
  integrated document — Step 2 now contains the complete system map (2.1–2.9: core entity/API,
  frontend surfaces, pricing tool, full backend consumer table, full frontend consumer table, DB
  tables, child-table precedent, permissions, migrations) rather than a ticket-scoped list plus a
  bolted-on addendum. All later steps (gap analysis, impact, design, tasks, edge cases, stakeholder
  questions, push-back) rewritten to reference the full picture natively.
