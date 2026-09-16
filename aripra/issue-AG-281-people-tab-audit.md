---
name: issue-AG-281-people-tab-audit
description: AG-281 — audit the People tab against the ticket's stated requirements; identify what's missing, what's present, and whether the ticket's own points are still valid against current code
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-16.** Analysis-only ticket, no code changes made (per explicit
instruction). Jira description NOT updated with the verdict — user didn't ask for that, only to mark
done in memory tracking.

User said "New ticket: AG-281 ... the People tab misses the mentioned things ... check what it
misses and what is present, and whether the mentioned points in the ticket are valid or not."
Explicit instruction: analysis only, no changes without permission.

Note: earlier in this same exchange the user initially said "AG-258" by mistake — that number is
already a closed ticket in memory (a narrow People-tab pivot column split) that turned out, on
fetching real Jira, to actually be the full parent "Parts Inventory — People tab" story (3
attribution axes: PO created by / Workshop manager / Technician, click-through, the already-built
90D+/No Date split, and an unresolved "does mechOwner really mean workshop manager?" business
question). User then corrected to AG-281 — a different ticket. Worth keeping in mind in case AG-281
turns out to be related to or overlapping with AG-258's still-open items.

**Fetched from Jira.** AG-281 (Bug, status "Reported", created 2026-09-08 by Shivansh Shukla) is
titled "AG-258 -> These things are missed according to the given requirement" — a QA flag against
AG-258's actual delivery (the full People-tab story found above), listing 6 specific claimed gaps:

1. Split the "90D+ / NO DATE" column into two separate columns (90D+, No Date).
2. Clicking a person's parts should open the exact task directly.
3. "PO Created By" should show the correct person (who actually created the PO).
4. "Technician" should show the correct technician (assigned to that task).
5. "Workshop Manager" should use `mechOwner` — NOT `mechCommentOwner`.
6. Backend needs a people-level-totals endpoint (per-person backlog calculation).

**Timing note:** this bug was filed 2026-09-08 — the day BEFORE I built and closed AG-258's own
90D+/No-Date split (2026-09-09).

**Verdict — all 6 items verified directly against current code, all already resolved:**
1. **90D+/No Date split** — `getPeopleAttributionPivot` (`inventory.service.ts:6222-6232`) computes
   `over90`/`noDate` as 2 separate SUMs; `people_tab.tsx:93,102-103` renders 2 distinct headers.
   Built by AG-258's own follow-up, 2026-09-09 (1 day after this bug was filed).
2. **Row click opens exact task** — `people_person_table.tsx:185`,
   `onClick: () => openTaskCard(row.original.taskId, row.original.vid)`. Already fixed by AG-272,
   closed 2026-09-03 — 5 days *before* this bug was even filed, suggesting the reporter was testing
   against a stale build/environment, not a real regression.
3. **"PO Created By" correct person** — `getPeopleAttributionPivot:6200` uses `po.poCreatedBy` (the
   actual PO creator), not `taskPart.createdBy` (a different person/moment) — correct, and notably
   *better* than what AG-258's own original Jira text had suggested reusing.
4. **"Technician" correct** — `getPeopleAttributionPivot:6201-6202` uses `task.assignedToUser`.
   Correct.
5. **"Workshop Manager" uses `mechOwner` not `mechCommentOwner`** —
   `getPeopleAttributionPivot:6206-6210` joins explicitly on `vehicle.mechOwner`; no reference to
   `mechCommentOwner` anywhere in the function. Correct, with the ticket's own explicit warning
   already respected.
6. **Backend people-level-totals endpoint** — `getPeopleAttributionPivot` itself IS this endpoint,
   returning `{name, parts, over90, noDate, value}[]` per person for all 3 roles. Already exists.

**Conclusion given to user:** ticket looks safe to close as "already resolved / not reproducible on
current code," most likely filed against a stale build. Verdict given in chat; not written back into
Jira (not asked). No code changes made — pure analysis ticket.

---

## HISTORY

- 2026-09-16: Ticket opened by user request ("New ticket: AG-281", corrected from an initial
  "AG-258" typo), memory file created immediately per [[create-ticket-file-immediately-on-open]],
  before any analysis.
