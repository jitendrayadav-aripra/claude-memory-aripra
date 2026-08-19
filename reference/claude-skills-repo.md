---
name: claude-skills-repo
description: "Location of the project's claude-skills repo (separate from the iOS app repos)"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 85508222-fb04-4af4-b07c-2b4d62f2d874
---

Project skills live in a separate repo: **`/Users/prakashpogra/Desktop/Projects/MikeStuff/claude-skills`**
(distinct from the app repos). **2026-06-18: this repo MIGRATED to GitHub `git@github.com:lumasoft-co/claude-skills.git`
(default branch `main`); re-cloned fresh (SSH), old GitLab clone deleted. Pull/fetch from GitHub now (NOT glab).
See [[repo-migration-github]].**

**Skills present (each at `<repo>/<name>/SKILL.md`) — refreshed 2026-06-18:** `github`, `gitlab`, `translate`,
`marketing-screenshots`, `blog-post`, `answer-reviews`, `lumabooth-brand`, `youtube-publish`, `sprint-planner`.
- **`github`** (NEW) = canonical MAP of the `lumasoft-co` org: every repo + one-liner + ALIAS RESOLUTION. Read it
  before picking a repo for a `gh`/`git` command or when a repo is named approximately. Key aliases: "claude-code"
  →`claude-skills`; "dslrbooth"/"Windows app"→`lumabooth_windows`; "iOS app"/LumaBooth→`lumabooth_ios`;
  "dashboard"/"web settings"→`lb-dashboard`; "site"→`dslrbooth_site`; "blog"→`dslrbooth_blog`; `*_preimport`=pre-
  migration snapshot, DON'T touch. Other repos to know: `github-tools` (GH automation / @luma support agent),
  `release-notes`, `lumabooth_android`. Regenerate map: `gh repo list lumasoft-co --limit 200 --json name,description,isArchived`.
- **`sprint-planner`** (NEW) = LumaBooth 3-app parity tracking & cross-repo sprint planning.

**IMPORTANT — these are NOT always registered as Skill-tool skills.** In some sessions `Skill(gitlab)`
returns "Unknown skill: gitlab" (the repo isn't wired into the session's skill set). FALLBACK when the
Skill tool can't load one: just **Read the `SKILL.md` directly from the path above and follow its steps
manually** (e.g. the `glab` commands it prescribes). Don't tell the user it's unavailable and stop —
read the file and do the work the skill describes.

**At the START of every new session, AUTOMATICALLY fetch + fast-forward pull this repo** (don't just
check) so I'm always on the latest skills before doing skill/workflow work:
```
git -C /Users/prakashpogra/Desktop/Projects/MikeStuff/claude-skills fetch -q
# if behind AND working tree clean -> fast-forward:
git -C .../claude-skills merge --ff-only @{u}
```
- **Tree clean + behind → pull (ff-only).** Tree **dirty** + behind → DON'T clobber: just flag it and let
  the user decide. Already up to date → say nothing / proceed.
- **Restart nuance:** an **updated existing skill** takes effect immediately — its `SKILL.md` is read at
  invoke time, so after the pull I follow the latest file as-is, no relaunch. A **brand-new skill** (new
  folder) only auto-registers into the Skill-tool list at Claude Code **launch**, so it surfaces next
  session — but I can still `Read` its `SKILL.md` and follow it manually this session if pointed at it.
- (For a hard guarantee independent of whether the session-start routine fires, a SessionStart hook in
  settings.json — or a shell alias that pulls before launching Claude Code — is the bulletproof mechanism;
  memory alone only reminds.)

The user updated the skill on 2026-06-10 (FYI, no action taken). Check this repo when a task involves
a custom skill/workflow rather than app code.

**Constraint (standing):** no secrets in the skills repo — the App Store Connect `.p8` API key stays in
the iOS repo, never committed here. See [[mac-screenshots-1934-state]] for the automation context.
