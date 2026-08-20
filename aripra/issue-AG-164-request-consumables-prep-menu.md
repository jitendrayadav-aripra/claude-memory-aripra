---
name: issue-AG-164-request-consumables-prep-menu
---

## NOW

- **Status (2026-08-20): DONE (closed by user).** Ticket closed at the user's explicit instruction.
  Build-time status as of 2026-07-22, kept for reference: backend + frontend both COMPLETE,
  `tsc --noEmit` clean on both repos. Two new migrations were written but NOT run as of that date;
  no manual browser verification had been done in that session either. Whether the migrations have
  since been run or the feature verified in-browser was not re-checked before closing — if this
  area comes up again, verify current DB/migration state rather than assuming either is done.
- **Correction made mid-build:** re-checked live DB before touching migrations (per user's
  explicit instruction) and found the original Request V2 create-table migration
  (`1784703539303-...`) had actually already been run — falsifying the earlier assumption it was
  still editable in place. Wrote a proper new `ALTER TABLE` migration instead. Also verified the
  new migrations' genuinely-captured timestamps (smaller than the DB's highest applied,
  `1787008000000` — one of `tasks/lessons.md`'s own documented mis-dated migrations) create no
  functional conflict with TypeORM's migration runner. Full reasoning in `tasks/todo.md`.
- **Jira:** AG-164 (parent epic AG-20), dependency AG-162 (`Consumables Request` permission,
  status "Ready for UAT"). Pulled both via Atlassian MCP — AG-164 covers only the technician
  submit surface; queue/approval is a separate ticket, but user directed reusing the already-built
  "Consumable Requests V2" tab (see [[issue-request-v2-consumables-flow]]) for approval instead of
  the original `consumable_transaction` pipeline AG-164's own dev notes describe.
- **Full plan lives in the repo:** `d:\aripra\projects\tasks\todo.md`, section "Request Consumables
  — dedicated Prep-menu submit surface (AG-164)". Read that for the full backend/frontend
  checklist; this ticket is the resume-pointer + decisions.
- **Feature summary:** new "Request Consumables" item under the Prep sidebar menu — technicians/
  bodyshop users submit a free-form consumable need (no product picker, no task/VRM link).
  Lands in the same `task_part_request_v2` table as the Task-Card Request V2 flow, reviewed in the
  same "Consumable Requests V2" Inventory tab, but with "Order" hidden (no task to attach a
  `task_part` to) — only Approve/Deny apply. Requester sees their own submissions'
  Pending/Approved/Denied status in a new self-view list on the same page.
- **Key decisions confirmed with user 2026-07-22:**
  1. Do NOT remove the original "Use Consumable" tile (`request_part_type_modal.tsx`) — AG-164
     calls for this but it's explicitly out of scope for this pass; everything stays additive.
  2. Fix a real, already-shipped permission bug found while researching this: confirmed via
     `user.service.ts` `getUserWithPermissions` (lines 817-840) that `related_permission` is a
     functional auto-grant, not cosmetic. `INVENTORY_CONSUMABLES_REQUEST`'s migration
     (`1784291486894-...`) sets `related_permission='INVENTORY_CONSUMABLES'`, meaning granting it
     today silently also grants the full Consumables tab — contradicts AG-162 AC4 explicitly. Fix:
     new migration nulls `related_permission` for that one permission row only (reversible
     `down()`), doesn't touch the 3 sibling consumables permissions whose parent-link is correct.
- **Next action:** none pending from me — remaining work is the user running the two new
  migrations and doing a manual browser pass (checklist in `tasks/todo.md`'s Verify section,
  including specifically re-testing that the permission-bug fix took effect).
- **New backend surface:** `GET /request-v2/my-requests` (self-view), `POST /request-v2` now
  Joi-validated and accepts an optional `taskId`.
- **New frontend surface:** Prep sidebar item "Request Consumables" →
  `/dashboard/request-consumables` (new page, reuses `AGRequestV2Modal` unmodified).
- **Don'ts:** don't touch `request_part_type_modal.tsx` (decision #1); don't add server-side
  permission enforcement on `/request-v2/*` (deferred to AG-125 by both tickets); don't build a
  new queue/approval UI — reuse `consumables_request_v2_tab.tsx` as-is plus the Order-hiding tweak;
  don't edit the 3 sibling consumables permission migrations, only
  `INVENTORY_CONSUMABLES_REQUEST`'s.

---

## HISTORY

- 2026-07-22 — User gave a chat-only description of this feature, framed as "reuse the existing
  Request V2 form/tab." Pointed to AG-164 for more context. Pulled AG-164 + its dependency AG-162
  via the Atlassian MCP tools (cloudId `761f8a59-91ff-493b-ab27-a38827186c27`) rather than working
  from the chat description alone — surfaced two things the chat didn't mention: AG-164 calls for
  removing the original "Use Consumable" tile, and cross-referencing AG-162's AC4 against the
  actual `related_permission` mechanism (verified via an Explore-agent research pass reading
  `user.service.ts`) revealed a live permission bug unrelated to but blocking correct reuse of
  `INVENTORY_CONSUMABLES_REQUEST` for this new surface. Presented full understanding + a 2-question
  AskUserQuestion for both discoveries; both resolved with the recommended option. Wrote the full
  implementation plan to `tasks/todo.md`. Implementation not started as of this entry.
