---
name: issue-AG-289-part-note-panel-status-arrow-info-icon
description: AG-289 — Part Returns Audit note panel header gains an origin→resolution arrow and an outcome-specific info icon
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-10.** `tsc` clean both repos, `next lint` clean on both changed
frontend files. Closed by explicit instruction. No migration. Ticket had no Jira description —
user gave requirements directly + a screenshot; title/description written into Jira afterward via
`editJiraIssue` (Sub-task under AG-255, created moments before this ticket was opened with me).

Two asks: (1) arrow between origin status and resolution status in the note panel header
("Incorrect → Refitted"), (2) info icon next to the resolution status badge with outcome-specific
detail (Resold → resale price, Refitted → destination task/vehicle, Listed → platform + price,
Scrapped → no icon at all).

**Traced the real source** — the two badges shown in the screenshot aren't rendered inside
`PartNotePanel` itself (the component the screenshot might suggest); they're in the **parent**,
`non_conforming_sign_off_modal.tsx:83-103` (origin `part.status` badge always shown, resolution
`part.resolutionStatus` badge alongside when present).

**Gap found for the Refitted case:** `NonConformingPart` type (`types/inventory.ts`) has no
`refittedVehicleId`/`refittedTaskId` fields at all yet — the comment there still reads "No
vehicle/task attachment yet — deferred," stale since AG-261 actually built that attachment. The
backend list query (`taskPartsNonConformingSection`, `inventory.service.ts:791`) has an **explicit**
`.select([...])` column list (not a bare `.getMany()`) that doesn't include those 2 columns either.
AG-261's own display-text batch-fetch was built into `getTasks` (Task Card's endpoint) — a
completely different query from this one (Part Returns Audit's own list), so it doesn't cover this
screen at all. Needs its own batch-fetch here, mirroring the exact `listedByUser`/`refittedByUser`
pattern already established in this same function (plain-int columns resolved via a separate
`UserRepository.find(In(...))` batch-fetch, not a join) — same idea, but resolving
`refittedVehicleId`→VRM and `refittedTaskId`→task category name instead of a user.

**Plan:**
- Backend: add `refittedVehicleId`/`refittedTaskId` to the select list; new batch-fetch (Vehicle by
  `In(refittedVehicleIds)` for VRM, Task+taskCategory by `In(refittedTaskIds)` for category name);
  attach `refittedVehicleVrm`/`refittedTaskCategoryName` onto each returned part.
- Frontend: `NonConformingPart` type gains the 4 new fields (+ fix the stale comment).
  `non_conforming_sign_off_modal.tsx` header — insert a small "→" between the two badges when
  `resolutionStatus` is set; add an MUI `Tooltip`+`InfoOutlinedIcon` next to the resolution badge
  (same visual pattern already established in `task_parts_status_badge.tsx` for the Task Card),
  content branching on `resolutionStatus`: Resold → resale price, Refitted → "Refitted to: {vrm} ·
  {category} (#{taskId})" (same phrasing as the Task Card's own tooltip line, AG-261), Listed →
  platform + asking price, **Scrapped → icon omitted entirely** (explicit ask).

No migration — all source columns already exist.

---

## HISTORY

- 2026-09-10: Ticket opened by user request ("new ticket: AG-289"), memory file created immediately
  per [[create-ticket-file-immediately-on-open]] before any analysis.
- 2026-09-10: Traced the real source (parent modal, not `PartNotePanel` itself) and found the
  Refitted-detail gap — `NonConformingPart` type and the list query both predate AG-261's
  vehicle/task attachment. Explained understanding, drafted title/description, asked permission.
  User approved — built both repos (backend batch-fetch mirroring the existing `listedByUser`
  pattern; frontend arrow + Tooltip/InfoOutlinedIcon on the resolution badge). `tsc` clean both,
  `next lint` clean on both changed frontend files. Gave short commit messages proactively.
- 2026-09-10: User asked to write the drafted title/description into the real Jira ticket —
  done via `editJiraIssue` (confirmed AG-289 is a Sub-task under AG-255, created just before the
  user opened it with me). User then said to mark AG-289 done and move to a new ticket.
