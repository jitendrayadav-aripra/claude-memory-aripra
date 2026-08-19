---
name: session-start-routine
description: "What to orient on at the start of every session / when resuming a ticket — runs automatically, no reminder needed"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 949bc40a-0319-4fca-a1a5-306f086b798d
---

At the start of each session (and when resuming a ticket), orient automatically — the user should not have to remind you:

1. **Conventions are already loaded** — the MEMORY.md index is injected each session; treat every line in it as a live standing rule (commit format, reuse, token-economy, check-recent-commit, GitLab reply rules, etc.).
2. **Auto-pull skills, then scan the available-skills list.** FIRST `git fetch` the claude-skills repo and **fast-forward pull when the tree is clean** (if dirty + behind, flag — don't clobber) so I'm on the latest; see [[claude-skills-repo]] for the exact commands + the restart nuance (updated skill = follow as-is now; brand-new skill = surfaces next launch). Then scan the session's available-skills list (in the system-reminder) and prefer the matching skill for the task. **Repos are on GitHub now (2026-06-18) → use `gh`, NOT glab/`gitlab` skill;** the `github` skill = repo map/alias resolution. Also `translate`, `marketing-screenshots`, `sprint-planner`, and the support/triage set (`google-cloud-logs`, `fotoshare-db`, `intercom-triage`, `rollbar-fix-errors`, `datadog-fix-errors`, `answer-reviews`). NOTE: the lumasoft custom skills are often NOT registered as Skill-tool skills — Read their `SKILL.md` directly from the claude-skills repo and follow it. If a skill our memory relies on is ABSENT, flag it rather than improvising.
3. **For a named ticket** ("#NNNN" / "load memory for #NNNN"), read that ticket file's `## NOW` block first; pull `## HISTORY` only if the task needs deep context.
4. **Apply token economy from the first action** (see [[token-economy]]) — don't re-explore or re-read big files to rebuild state that memory already holds.
5. **Debug-logs trigger is live from session one** — if the user says **"ready for the debug logs"** or ANY variant ("check/see/show the debug logs", "check the logs", "debug logs", etc.) after building a fix in Xcode, immediately read this run's logs with NO path/paste asked: auto-detect the target (Mac container / booted sim / connected device) and pull `GoogleCloudLogEntries.jsonl` per [[read-google-cloud-logs]], consult [[log-signal-map]] to map the lines → active screen/emitting code, and surface only what's tied to the change (append any newly-traced line to the map). This is a standing convention, not a one-off — works in any new session/login on this Mac, no reminder needed.
6. **Own context hygiene from session one** — I proactively manage chat-context cost without being asked (see [[token-economy]] #9): keep the active ticket's NOW block current as we go, and at every ticket boundary (done/parked) or after a heavy read (big file / large log·JSON / long thread) save the NOW block then offer a one-line `/compact` (mid-ticket) or `/clear` (between tickets). State lives in memory, so the chat is disposable once written. Applies in every new session/login, no reminder needed.

**Why:** the user has explicitly said they will not re-remind these; reliable auto-orientation is what makes memory worth keeping and keeps token/time cost low. Extends [[token-economy]] and [[github-reply-tone]] (post via `gh`, not glab).
