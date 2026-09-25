---
name: issue-AUT-3639-booking-link-multiple-vehicles
---

## NOW

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
- **Next action:** none — both documents fully reconciled and self-contained. Two highest-leverage
  open items unchanged: (1) scoping decision on which of the ~20 newly-surfaced consumer files are
  in-scope vs. fast-follow (now tier-organized — Step 8 Q9), (2) per-tier product content decisions
  (WhatsApp/Audiofy/email — several need sign-off from whoever owns those integrations, not just
  the client). Awaiting user/stakeholder direction before any implementation.
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
