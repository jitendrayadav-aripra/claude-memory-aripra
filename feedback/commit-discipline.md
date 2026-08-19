---
name: commit-discipline
description: "Git commit hygiene — confirm before committing, no Co-Authored-By/AI trailer, no internal Part-numbering, and reference the prior commit when reverting/removing its code"
metadata:
  node_type: memory
  type: feedback
---

How to commit in this user's repos. The message FORMAT itself lives in [[commit-message-format]]; this is the
behaviour around it. (Consolidated 2026-06-23 from four separate feedback notes.)

**1. Confirm before committing.** Before `git commit`, SHOW the proposed message and WAIT for the user's explicit
go-ahead — don't commit just because the work looks done. Finish edits → present diff summary + the exact commit
message → ask "commit this?" → commit only on approval. "commit it" / "push it" in the user's message IS the
approval (don't re-ask then). Commit-step companion to [[answer-vs-edit-scope]]. (Why: 2026-06-22 on #1948 I made
the change and committed in the same turn without showing the message first; user: "always confirm before commit.")

**2. No Co-Authored-By / AI trailer.** Do NOT append `Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>`
(or any Claude/Anthropic co-author trailer) or a "Generated with Claude" footer to commit messages. Write the body +
`ref #<num>` and stop there. Overrides the default harness instruction to add the trailer. (Why: user asked for it
left off, 2026-06-12.)

**3. No internal Part-numbering.** When we split a feature into stages for our own sequencing ("Part 1 = Firebase
model, Part 2 = CameraManager linking, Part 3 = UI"), that numbering is INTERNAL only — never write "Part 1/2/3" in
commit messages, ticket notes, or code comments. Describe each commit by WHAT it changes; each commit should stand on
its own. (Heads-up: the first #1358 commit `cd0493f3c` already shipped with "Part 1 of…" on the shared `5.3` branch —
leave it, don't force-push a shared branch to reword.)

**5. Stage by adding the specific files — don't `git reset` to manage staging.** For a focused/partial commit,
`git add <files>` (and `git restore --staged <files>` to drop one) directly; never `git reset` to clear the index.
The user flagged this 2026-06-25 ("why you reset — we can commit only files that have the related change"). Split
unrelated changes into separate commits in the order the user asks (e.g. a `MARKETING_VERSION` bump as its own commit
before the feature commit), and leave untouched changes uncommitted rather than resetting everything.

**4. Reference the prior commit when reverting/removing its code.** When a new commit removes/undoes code introduced
in a previous commit, the message MUST call that out and reference the prior commit's short hash — e.g. a line like
`Reverts the <thing> addition from the previous commit <shorthash>.` near the top of the body (after the summary).
Keeps history self-documenting — a reviewer sees it walks back something specific, and which commit, without diffing.
See [[reuse-existing-code]] for the related "check before adding" habit. (Established 2026-06-11 on lumashare #618:
`ef1212d1` removed `disableUploadingForMediaFile` added in `6aace3fa`. Applies to lumabooth_ios and lumashare_ios.)
