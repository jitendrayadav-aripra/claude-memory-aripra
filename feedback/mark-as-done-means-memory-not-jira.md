---
name: mark-as-done-means-memory-not-jira
description: "mark as done" / "mark the ticket as done" means close it in MY memory (MEMORY.md → ARCHIVE.md), never transition the actual Jira ticket status — that needs separate explicit permission every time
metadata:
  type: feedback
---

When the user says "mark [ticket] as done" (or equivalent), by default that means:
- Move its line from `MEMORY.md`'s active index to `ARCHIVE.md`.
- Update its memory file's NOW block to reflect DONE status.

It does **NOT** mean transitioning the real Jira ticket's status — never call
`transitionJiraIssue` (or any status-changing Jira write) on my own initiative in response to
"mark as done." Only touch the Jira ticket's status when the user **specifically and explicitly**
asks for that (e.g. "mark the Jira ticket as done too," "transition it in Jira," "close it in Jira").

**Why:** caught live on AG-280 (2026-09-16) — I transitioned the real Jira ticket
(Reported → In Review/QA) after "mark the ticket as done," which also auto-reassigned it to the
original reporter as a side effect of that workflow's transition. The user's memory-only intent was
completely different from what I did to their live Jira board, and Jira transitions in this project
can have real side effects (reassignment, workflow gates, QA/UAT steps) that shouldn't be triggered
speculatively.

**How to apply:** on any "mark as done," update memory only. Writing a title/description into Jira
(when a ticket is blank/placeholder, e.g. [[issue-AG-297-remove-legacy-waste-chart]]-style retroactive
tickets) is a separate, already-established behavior and not itself a status change — that's fine to
keep doing when it was asked for. But actually moving the Jira **status** field requires a distinct,
explicit ask, every single time — never bundle it into "mark as done" by default again.
