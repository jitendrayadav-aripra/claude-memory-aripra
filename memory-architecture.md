---
name: memory-architecture
description: "How this memory system is laid out and how to use it — read this to understand the file structure, what loads when, the cheap→expensive read order, the ticket lifecycle, and the enforcement hook"
metadata: 
  node_type: memory
  type: reference
  originSessionId: 4589d9a8-73ce-4446-975d-859b10b7769e
---

Quick orientation for any new session or agent: this is how the memory store is organized and used.
The whole point is token economy — pay a small fixed cost each session, load detail only on demand.

**Location (since 2026-06-22):** canonical store at `~/Desktop/Projects/MikeStuff/memory/`, shared across ALL
Lumasoft projects and loaded from ANY directory via global `~/.claude/CLAUDE.md` (+ symlinks from the old
per-project slug dirs, so harness auto-memory and future saves both land here). **Global rules = type folders**
(`feedback/`, `reference/`, `convention/`, `pattern/`, clean prefix-less names — reorganised 2026-06-23); **root =
index/loader files** (this index, SPEC, `ARCHIVE.md`, `README.md`, `worklog-daily`, `memory-architecture`);
**`<project_name>/` subfolders = per-project tickets** (`lumabooth_ios/`, `lumashare_ios/`, …). So "back to #N"
resolves from any repo or the MikeStuff parent.

```
                       MEMORY ARCHITECTURE (cross-project, MikeStuff/memory)
═══════════════════════════════════════════════════════════════════════════════════

  NEW SESSION ──────────────► loads ONLY:  MEMORY.md  (~2.3k tok, every session, ANY cwd)
                                            │
        ┌───────────────────────────────────┼───────────────────────────────────┐
        │                                   │                                   │
   ┌────▼─────┐                      ┌──────▼───────┐                    ┌──────▼──────┐
   │  SPEC    │  rules auto-enforced │ CONVENTIONS  │  live operating    │   ACTIVE    │
   │  header  │  · 1 line/memory     │ & references │  rules (commit fmt,│   TICKETS   │
   │          │  · NOW≤400, read-NOW │  (hooks into │  token economy,    │  pending-   │
   │          │  · token gate        │   *.md files)│  reply tone, …)    │  on-ME only │
   │          │  · lifecycle/resume  │              │                    │             │
   └──────────┘                      └──────────────┘                    └─────────────┘
                                                                               │
═══════════════════════════════════════════════════════════════════════════════════
   ON-DEMAND (NOT loaded each session — zero idle cost):
                                                                               │
   "back to #N" / "load memory for #N"  ──► glob */issue-<N>-*.md  BY NUMBER  ◄──┘
        (under <project_name>/ — works whether ACTIVE or ARCHIVED, index-independent)
                                   │
                                   ▼
                    ┌──────────────────────────────┐      ┌───────────────────────────┐
                    │  issue-<N>-<slug>.md          │      │  ARCHIVE.md (browse list)  │
                    │  ┌── ## NOW  (≤400 tok) ◄─────┼──────┤  thin pointers for         │
                    │  │   status·branch·SHAs·       │ read │  To Test/Done tickets:     │
                    │  │   decisions·don'ts          │ first│  status + commit refs + gh │
                    │  ├── ---                       │      └───────────────────────────┘
                    │  └── ## HISTORY (detail) ◄──── pull only if NOW insufficient
                    └──────────────────────────────┘
                                   │  still need more? escalate CHEAP ► EXPENSIVE
                                   ▼
                    git show <commit ref>   (local, cheap, has root-cause body)
                                   │
                                   ▼
                    GitHub issue thread     (LAST RESORT — filter to 1 comment, never dump)

═══════════════════════════════════════════════════════════════════════════════════
   LIFECYCLE (bidirectional)            WRITE-BACK (after work, automatic)
   ─────────────────────────            ──────────────────────────────────
   pending-on-me                        • keep NOW current, prune → HISTORY
      MEMORY.md  ───To Test/Done───►     • append worklog-daily.md (all work)
      ACTIVE              ARCHIVE.md     • new lesson? grep memory first, extend
        ▲   ◄──bounced back (Mike)──┘    • ticket hits To Test → move to ARCHIVE.md

═══════════════════════════════════════════════════════════════════════════════════
   ENFORCEMENT & AUTOMATION (hard hooks in ~/.claude/settings.json — deterministic, unlike soft memory)
   ────────────────────────────────────────────────────────────────────────────────────────────────
   PreToolUse(Bash) hook  └─► BLOCKS tail/cat/head GoogleCloudLogEntries.jsonl (no grep); ALLOWS grep -F <marker>
   SessionStart    hook   └─► auto fetch + ff-only pull of the claude-skills repo each launch
                               (~/.claude/hooks/pull-skills.sh — clean-tree only, never clobbers; exits 0)

═══════════════════════════════════════════════════════════════════════════════════
   SKILLS & SLASH COMMANDS (live in a separate repo + ~/.claude — NOT in MEMORY.md → zero per-session cost)
   ────────────────────────────────────────────────────────────────────────────────────────────────────
   claude-skills repo (auto-pulled) ─► translate · github · google-cloud-logs · fotoshare-db · intercom-triage · …
        task match / typed "/skill"  ─► run the skill FIRST (it's the verified path); else Read its SKILL.md
        for log tickets: the google-cloud-logs skill is PRIMARY; [[read-google-cloud-logs]] = manual fallback
   ~/.claude/commands/*.md ─► /changes /hours /backup /rt /ready /commands /logs /savings /memflow /artifact
        each = a THIN pointer to its memory file (single source of truth); read only when invoked
        backed up in this repo at commands/ (restore source); NO index line → zero idle cost
```

## How to read it

- **Every session** pays only for `MEMORY.md` (~2.3k tok): the auto-enforced SPEC header, the live operating
  rules/conventions, and the *active* ticket queue (tickets with a pending action on me).
- **Archived/done tickets cost nothing at idle.** Resume by ticket number: `issue-<N>-*.md` is found on disk
  whether the ticket is active or archived — no need to scan any index.
- **Reading escalates cheap → expensive:** ticket NOW block → its HISTORY → `git show <commit ref>` (local,
  carries the root-cause body because commit messages are written that way) → GitHub thread only as the LAST
  resort, filtered to the relevant comment. A curated local file always beats a full GitHub-thread read.
- **Lifecycle is bidirectional:** a ticket moves to `ARCHIVE.md` when it's To Test / Done / handed off, and moves
  back to the active list in `MEMORY.md` when Mike bounces it back.
- **Write-back is automatic** after work: keep the NOW block current (prune resolved detail into HISTORY), append
  `worklog-daily.md`, and before adding a NEW lesson grep existing memory and extend rather than duplicate.
- **Two things are hard-enforced** by `~/.claude/settings.json` hooks (deterministic, unlike soft memory): the
  `PreToolUse` token-gate (raw log dumps blocked) and the `SessionStart` hook that auto-pulls the skills repo each
  launch. Every other rule is the SPEC I follow.
- **Skills & slash commands sit OUTSIDE the per-session load** (separate repo + `~/.claude/commands/`), so they cost
  zero idle tokens. Skills are the verified path — run the matching skill first; for log tickets the
  `google-cloud-logs` skill is primary and [[read-google-cloud-logs]] is the manual fallback. Slash commands are thin
  pointers to memory files, backed up in this repo under `commands/`.
- **This file is the source of truth for the system.** When asked to explain or DIAGRAM how memory + skills work,
  read THIS file (and the MEMORY.md SPEC) and build FROM it — do NOT reconstruct the layout from scratch, or the
  picture drifts from reality.

## Files at a glance

At the ROOT (global — every project), index/loader files only:
- `MEMORY.md` — the index, loaded every session. SPEC + conventions/references (one line each) + ACTIVE tickets.
- `ARCHIVE.md` — To Test/Done tickets as thin pointers (status + commit refs + GitHub link). Loaded on demand.
- `worklog-daily.md` — dated record of ALL work (commits + non-commit), the source for "show my changes".
- `README.md`, `memory-architecture.md` — orientation docs.

Type folders (global rules — every project), clean prefix-less names (reorganised 2026-06-23):
- `feedback/<name>.md` · `convention/<name>.md` · `reference/<name>.md` · `pattern/<name>.md` — durable rules,
  recipes, patterns. The `name:` slug = the clean basename; `[[wikilinks]]` use that slug (e.g. `[[token-economy]]`).

Under each `<project_name>/` (per-project):
- `issue-<num>-<slug>.md` — one per ticket: `## NOW` (≤~400 tok) + `---` + `## HISTORY`. Loaded on recall.
  Index/ARCHIVE links point to `<project_name>/issue-<num>-<slug>.md`.

## Reusing this across projects / platforms / trackers + sharing with a teammate

The store is designed to serve MANY projects/clients/trackers from one system. Conceptually two halves:
- **GLOBAL (reusable anywhere, any tracker incl. Jira/GitLab):** the system itself (this doc, the `MEMORY.md`
  SPEC, `daily-worklog`+`worklog-daily`, `session-start-routine`, `session-end-backup-memory`, the skills infra)
  + the work-style rules (`commit-discipline`, `reuse-existing-code`, `check-recent-commit-before-tweaking`,
  `answer-vs-edit-scope`, `token-economy`, `debugging-method`, `verify-claims-before-posting`,
  `cross-reference-linked-tickets`, `prefers-conversational`, `work-hours-stop-signal`) + the communication
  structure/tone (`github-reply-by-ticket-type`, `github-reply-tone` — named "github" but tracker-agnostic).
  None of these depend on GitHub/iOS/Lumasoft.
- **SCOPED (per platform/org/project):** GitHub-specific (`github-command-cookbook`, the `ref #num` footer in
  `commit-message-format`, `branch-milestone-discipline`, `github-handles`, `github-bare-commit-refs`,
  `repo-migration-github`, `lumadot-agent`); iOS/LumaBooth-specific (`prefer-swift-over-objc`,
  `read-google-cloud-logs`, `log-signal-map`, `pull-lumabooth-event-firebase-json`,
  `asset-url-removal-shared-event`, `ipad-ui-scaling-swiftui-hosting`); and the per-project ticket folders +
  the worklog (a person's billable history).

**Marking scope = TAG, don't MOVE (zero-risk, zero idle tokens).** Add an optional `scope:` frontmatter field
(`global` | `<platform-or-client>`, e.g. `lumasoft-github`, `ios`). It is INERT: the per-session load reads
`MEMORY.md`, never each file's frontmatter, so a `scope:` tag changes NO existing automation (hooks, commands,
loader, `[[wikilinks]]`, paths all unchanged) and costs ZERO idle tokens. It is read only at bootstrap by
`setup.sh`, or when a human/agent wants to filter. Do NOT physically move/rename files to "separate" global vs
scoped — that would break index links, wikilinks, and hook paths. Tagging achieves the split with none of that risk.

**Sharing with a teammate (different client, GitLab/Jira) via `setup.sh`.** A `setup.sh` bootstraps the GLOBAL
half into a FRESH environment without touching the owner's repo: installs the wiring (a `~/.claude/CLAUDE.md`
templated to their clone path, `commands/*.md` → `~/.claude/commands/`, the hooks + `settings.json` entries from
`scripts/`), and seeds a `MEMORY.md` containing only `scope: global` rules + an empty project scaffold + a fresh
worklog. The teammate then adds their own scoped layer — a `glab`/Jira cookbook (Jira via the Atlassian Rovo MCP;
GitLab `glab` is close to the existing gh cookbook) + their client ticket folder. Because `setup.sh` writes only
to the new location and the `scope:` tags are inert, the owner's setup/automation/token footprint are unchanged.

**Topology for 2+ people / clients (confidentiality):** a shareable **core kit** (global files + `setup.sh` +
commands + hooks) vs each person's **private** ticket+worklog repo — so billable worklogs and client tickets
never cross between people. Global files can be shared/symlinked; client data stays separate.

**Sharing as a REPO without leaking history — `./setup-new-user.sh --core-repo <DIR>`.** You cannot share the
owner's master repo and expect it to expose only globals: git carries full history, so any past commit (tickets,
worklog) is recoverable, and a branch does NOT isolate. The supported answer is a **separate, dedicated core repo
that by construction only ever contains globals.** `--core-repo` builds the globals-only store (via `core_install
… no` — packaging mode, so it does **not** touch the owner's `~/.claude`), then `git init`s it with a CLEAN
history (nothing from master) and an initial commit; you push it to its OWN remote (e.g. `<you>/claude-memory-core`)
and share THAT. Re-running on the same dir REFRESHES globals (guarded by a `.core-repo` marker so it refuses to
commit into an unrelated existing repo). The manifest (`core/manifest-files.txt`) holds back every client/scoped
reference (handles, log maps, cookbooks, artifact `*-link.md`, repo-migration, lumadot) and ships only the generic
system files — verified: a core-repo build contains no client data and a fresh worklog.

**Status: IMPLEMENTED — two scripts (one per audience):** **owner** = `./setup.sh` (the full personal store) and
**new user** = `./setup-new-user.sh` (the guided wizard for anyone else / a new client). `setup.sh` is owner-only
and redirects `--init`/`--core` to the new-user script. `./setup-new-user.sh --core <DIR>` is its non-interactive
global-kit engine (CI / scripted handoff). All built on `core/` (tested sandboxed). The concrete "which is global"
mechanism is **manifests** — `core/manifest-files.txt` + `core/manifest-commands.txt` (chosen over
per-file `scope:` frontmatter for simplicity; the frontmatter tag remains an optional future refinement). `--core`
copies the listed global files + commands into a fresh store, seeds a global-only `MEMORY.md` from
`core/MEMORY.seed.md` + a fresh worklog/ARCHIVE, writes a generic `~/.claude/CLAUDE.md`, and prints the
client-specific hooks as a recommendation (not auto-installed). It backs up anything it would overwrite under
`~/.claude` and never mutates the source repo. Default `./setup.sh` (owner re-bootstrap) is unchanged. See
`core/README.md`.

**Interactive new-user wizard: `./setup-new-user.sh`** (a separate script from the owner's `setup.sh`) — prompts
client / tracker / repo / logs, lays the global kit (its `core_install` function, also exposed as `--core DIR`),
scaffolds the `<client>/` ticket folder + a tracker cookbook stub
(github=`gh` / gitlab=`glab` / jira=Atlassian MCP), then offers the right auth: it can launch the CLI logins
(`gh`/`glab`/`gcloud auth login`) when run in the user's own terminal, but **Jira (and any MCP connector) auth is
done INSIDE Claude Code** (`mcp__claude_ai_Atlassian_Rovo__authenticate`), not the shell — so the wizard instructs
that step rather than executing it. Run `--init` directly in a terminal (it prompts + may launch interactive auth),
not via the assistant.
