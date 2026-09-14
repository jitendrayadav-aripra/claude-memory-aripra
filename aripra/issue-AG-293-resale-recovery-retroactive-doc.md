---
name: issue-AG-293-resale-recovery-retroactive-doc
description: AG-293 — retroactive documentation ticket for the estimated resale recovery feature (computeSuggestedResalePrice), originally built under AG-270's research but never given its own build ticket
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-14.** Retroactive documentation ticket, no code change — the feature
itself was built earlier (2026-09-09, under AG-270's research ticket, which is itself still "research
only, not a build commitment" per its own description and never reflected the shipped work). Same
pattern as [[issue-AG-273-part-note-panel-resolution-actions]] (retroactive doc ticket for AG-260's
already-built work).

**Immediate trigger:** the feature was found missing from the current working branch (a
`AKASH/Feature-AG-271-AI-recoverable-resaleprice` branch had it, current branch didn't) — traced via
reflog that nothing was lost, it was simply sitting on its own unmerged branch. Cherry-picked the 2
clean, self-contained commits (`c3de101d4` backend, `8c06fcf81` frontend — confirmed via
`git log`/`git show` these were the only commits on that branch not already superseded by later
tickets rebuilt independently on the current branch) onto the current working branches:
- Backend (`AKASH/CR-AG-284-2-addnewcolumns-forresolution-stagebuckets`): clean cherry-pick, no
  conflicts → commit `ad43afc80`.
- Frontend (`AKASH/Feature-AG-292-Overview-tab-add-hoverexplanations-to-KPIcards`): 4 conflicts
  (`overview_tab.tsx`, `task_parts_status_badge.tsx`, `create_task_modal_new.tsx`, `types/task.ts`)
  — all genuinely additive against AG-261/274/282/288's later work on the same lines, merged both
  sides rather than picking one → commit `b5a972dd7`. `tsc`+`next lint` clean both repos after.
- Neither commit has been pushed by me at any point — confirmed to the user explicitly after a
  question about visibility (git status showed "ahead of origin by 1 commit", not a push). User
  pushed both via Sourcetree themselves after confirming it was a plain, non-force push to their own
  feature branches only.

**The feature itself — `computeSuggestedResalePrice()` (`non_conforming_tab.tsx`):**
anchor = `invoiceUnitPrice ?? poUnitPrice`; starting bracket 50% (Faulty) / 80% (Incorrect, Not
Required); decays 5 percentage points per 30 days since arrival; floors at 20% regardless of age.
- **Overview tab** — "Awaiting eBay Listing" tile headlines the estimated recoverable value instead
  of price paid; price paid moved to a hover tooltip. Backend: `getPartsResolutionBuckets` gained
  `readyForSale.recoverableValue`, same formula mirrored in SQL (kept in sync manually).
- **Task Card** — status badge's info-icon tooltip gained a "Recoverable: £X" line for
  Faulty/Incorrect/Not Required parts only.
- **Caveat (from the original commit's own comment, still true)**: percentages are placeholder
  defaults, not validated against real sold prices.

**Related:** [[issue-AG-270-ai-resale-price-research]] (the research ticket this documents — still
open/in-progress, research-only, never reflects this shipped build).

---

## HISTORY

- 2026-09-14: User asked whether merging vs. cherry-picking a specific old branch
  (`AKASH/Feature-AG-271-AI-recoverable-resaleprice`) was the right way to bring this feature back —
  gave an opinion (cherry-pick 2 clean commits, not a full branch merge, since ~20 commits of drift
  were mostly already superseded on the current branch), grounded in actual `git log` history rather
  than guessed. User then discovered they were sitting ON that old branch (checked out earlier to
  explore) — traced via reflog to confirm no session work was lost, just an intentional branch
  switch, before answering the reversed merge-direction question. User checked back out, said "go
  for cherry pick," cherry-picks executed and conflicts resolved as described above.
- 2026-09-14: User asked why no changed files were visible to push — clarified that
  `git cherry-pick` commits directly (no pending-file-diff state), confirmed via `git log`/`git
  status` that both repos were "ahead by 1 commit," neither pushed nor merged by me. User then
  confirmed via a Sourcetree screenshot showing the exact same commit hash, pushed both branches
  themselves.
- 2026-09-14: User asked for a new ticket documenting this work specifically, referencing AG-270.
  Fetched AG-270's real Jira state (Sub-task, parent AG-260, still "In Progress"/research-only) to
  ground the draft rather than guess. Proposed title+description; user created the ticket as AG-293
  with that text verbatim, then asked to mark it done + update the living doc.
