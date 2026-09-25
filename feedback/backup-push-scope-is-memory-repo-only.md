---
name: backup-push-scope-is-memory-repo-only
description: "/backup" or "push it" (memory context) always means commit+push this store to jitendrayadav-aripra/claude-memory-aripra — never touches carplanet or car-planet-backend repos/branches
metadata:
  type: feedback
---

When the user says `/backup`, "push it", or "commit it" in the context of memory work, that
means: `git add -A` / commit / push inside `claude-memory-core-store` only, targeting remote
`jitendrayadav-aripra/claude-memory-aripra`.

It never means committing or pushing anything in the `carplanet` (frontend) or
`car-planet-backend` repos, or touching any branch there — those are separate project codebases
with their own commit-discipline rules ([[commit-discipline]] still applies to them too, but
that's a distinct, separate approval, never implied by a memory-repo "push it").

**Why:** user explicitly clarified this to prevent cross-repo ambiguity, since a session often has
both the memory store and the project repos in play at once.

**How to apply:** before running any git command in response to "/backup"/"push it"/"commit it",
confirm the working directory is `claude-memory-core-store` and the remote is
`jitendrayadav-aripra/claude-memory-aripra`. If the user's intent could plausibly be about a
project repo instead, ask — don't default to the memory repo silently, and never default to a
project repo either.
