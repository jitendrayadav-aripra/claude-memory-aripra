# Memory index

> ## SPEC — auto-enforce every session, no reminder (full detail: [memory-architecture.md](memory-architecture.md))
> 1. **Index = ONE line per memory** (a hook, never content); detail lives in the linked file.
> 2. **Ticket shape:** `## NOW` (≤~400 tok: status, branch+refs, next action, decisions/don'ts, constraints)
>    `---` `## HISTORY`. On recall **read NOW only first**; pull HISTORY only when deep context is needed. Keep
>    NOW current; prune resolved gotchas into HISTORY. (Works for any tracker — key files by the tracker's id,
>    e.g. `issue-<num>-*.md` for GitHub/GitLab, `issue-<KEY>-*.md` for Jira.)
> 3. **Token gate (in + out):** never `tail`/`cat` raw logs/JSON into chat — `grep -F` the ticket marker, cap
>    ~10 rows; announce big reads, let the user narrow. See [[token-economy]].
> 4. **Before a NEW lesson, grep existing memory — extend, don't duplicate.**
> 5. **Lifecycle:** this index lists only tickets with a **pending action on me**. To Test / Done / handed off →
>    move the line to `ARCHIVE.md`; bounced back → move it back here.
> 6. **Resume (index-independent):** on "back to #<id>" / "load memory for #<id>", glob `*/issue-<id>-*.md`
>    (active OR archived), read its NOW. REPORT FIRST (phase·branch·refs·next), then STOP and ask — don't resume
>    coding unprompted. See [[answer-vs-edit-scope]].
> 7. **Layout — rules GLOBAL (foldered by type), tickets PER-PROJECT.** Rule files: `feedback/`, `reference/`,
>    `convention/`, `pattern/` with CLEAN prefix-less names; `name:` slug = basename, `[[wikilinks]]` use that
>    slug. Index/loader files at ROOT (this index, `ARCHIVE.md`, `README.md`, `worklog-daily.md`,
>    `memory-architecture.md`); tickets under `<project_name>/`.

> **⚙️ SET UP YOUR TRACKER/PLATFORM (do this first):** add a `reference/<tracker>-command-cookbook.md` for your
> tracker — GitHub (`gh`), GitLab (`glab`), or Jira (Atlassian Rovo MCP) — and note your repos/handles. Add
> platform reference files (logs, build) as you need them. The rules below are tracker-agnostic; anything
> tracker-specific you add is yours. See [memory-architecture.md](memory-architecture.md) → "Reusing this across
> projects" for the global-vs-scoped model.

> **New here?** Read [memory-architecture.md](memory-architecture.md) for the layout, what-loads-when, the
> cheap→expensive read order, the ticket lifecycle, the hooks, and the skills/slash-commands layer.

## Conventions & preferences

- [Commit message format](convention/commit-message-format.md) — `type(scope): summary` + a ticket-ref footer (adapt the footer to your tracker: GitHub/GitLab `ref #num`, Jira smart-commit `KEY-123`); body paragraph for root-cause on non-trivial fixes
- [Commit discipline](feedback/commit-discipline.md) — confirm before `git commit` (show msg, "commit it"/"push it" = approval); NO Co-Authored-By/Claude footer; a revert/removal commit cites the prior short hash
- [Prefers conversational Qs](feedback/prefers-conversational.md) — ask inline in prose, avoid pick-lists
- [Reuse existing code](feedback/reuse-existing-code.md) — grep for an existing helper before writing; reuse over reinvent (applies to memory too)
- [Check recent commit before tweaking](feedback/check-recent-commit-before-tweaking.md) — on "tweak what we just did", read the commit/`git diff` FIRST
- [Answer ≠ edit (stay in scope)](feedback/answer-vs-edit-scope.md) — "answer"/"draft a reply"/"explain" = read-only; find a needed change → report + ask, don't edit unprompted
- [Token economy](feedback/token-economy.md) — grep/offset never whole files; subagent for broad searches; batch edits; never dump raw logs/JSON (grep the marker)
- [Debugging method](feedback/debugging-method.md) — MEASURE FIRST: auto-add a `#<ticket> TEMP DEBUG` logger capturing all candidates in ONE pass, auto-read, AUTO-CLEANUP before commit; lock the repro matrix; deterministic fix
- [Session-start routine](feedback/session-start-routine.md) — auto-orient: index = live rules; scan skills; read ticket NOW first; token economy from action one
- [Session-end: back up memory](feedback/session-end-backup-memory.md) — at wrap-up / "back up", commit + push the memory repo (keep it PRIVATE, owner-only)
- [No library install without permission](feedback/no-library-install-without-permission.md) — never `npm install` a new dependency (either repo) on my own initiative, even when obviously the right tool — always ask first
- [Ticket reply BY TYPE](feedback/github-reply-by-ticket-type.md) — classify (log-investigation / new-requirement / existing-code-change) → use that structure; conclusion-first, annotated evidence, symptom-vs-root-cause, end with an ask (tracker-agnostic)
- [Ticket reply tone](feedback/github-reply-tone.md) — write in your own human voice, not AI-sounding; "we" not "I"
- [Support-ticket comment structure](feedback/support-ticket-comment-structure.md) — log-investigation reply: version check → log lines → cases → if unproven say "replicating"; query by device id
- [Verify claims before posting](feedback/verify-claims-before-posting.md) — verify each claim against the actual logs/`git diff` line-by-line; PROVES vs CONSISTENT-WITH
- [Cross-reference linked tickets](feedback/cross-reference-linked-tickets.md) — when a ticket references another already in memory, auto-pull that NOW block before acting
- [Daily worklog → Excel](convention/daily-worklog.md) — "show my changes"/"EOD log" → READ [[worklog-daily]] for the date; output JUST a plain Details block; mandatory cheap git cross-check (oneline only)
- [Worklog data (ALL work)](worklog-daily.md) — single source of truth; APPEND every session for commits AND non-commit work
- [Work-hours stop-signal](feedback/work-hours-stop-signal.md) — if you bill hourly: `/hours` derives honest daily hours from [[worklog-daily]], NEVER pad, and proactively say "that's a full day, good to stop"
- [Don't default ticket fields to N/A](feedback/dont-default-ticket-fields-to-na.md) — check each ticket has NO real surface for a template field (esp. "Notes for Tester") before copying N/A from a sibling ticket
- [Segmented control, one-sided radius](feedback/segmented-control-one-sided-radius.md) — one-sided border-radius per segment in given CSS = seamless multi-segment bar (round only outer ends), not a padded floating-pill toggle

## References

- [claude-skills repo](reference/claude-skills-repo.md) — optional shared skills repo; auto fetch + ff-only pull at session start
- [Session-start skills hook](reference/session-start-skills-hook.md) — `SessionStart` hook auto-pulls the skills repo each launch (adapt the path/repo to yours)
- *(add your tracker cookbook + platform reference files here)*

## Tickets — ACTIVE (pending action on me). Done/To Test → [ARCHIVE.md](ARCHIVE.md).


