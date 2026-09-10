---
name: issue-AG-285-scrapped-value-total
description: AG-285 — add a 5th "Scrapped" box/segment to the Overview "Parts Pending Return or Resale" widget, currently the only resolution outcome with no total shown anywhere
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-10, migration N/A.** `tsc` clean both repos, `next lint` clean on
all 3 changed frontend files. Closed by explicit instruction.

Scrapped parts were completely invisible on the Overview "Parts Pending Return or Resale" widget —
every bucket query excludes `resolutionStatus IN (Resold, Refitted, Scrapped)`, so a scrapped
part's value vanished from the dashboard entirely rather than surfacing anywhere. Fetched via Jira
(Atlassian Rovo MCP, reconnected this session) — ticket had unusually detailed developer notes
already researched against an earlier state of the branch; re-verified every reference against the
current code before building since line numbers had drifted from AG-270/AG-284/AG-261 work since.

**Key finding that shaped the implementation:** confirmed directly from `markTaskPartAsScrapped`
(line ~3709) that Scrapped applies to **any origin status** — no restriction to Faulty/Incorrect/
Not Required, unlike Refitted. Also checked `markTaskPartAsListed` (a part must be listed before it
can be scrapped) and confirmed it has no origin-status gate either. This meant the existing shared
`taskPart.status IN (Faulty, Incorrect, Not Required)` filter (applied once, up front, to all 4
existing buckets in one query) could NOT just get a 5th `CASE` arm — that would silently
under-count Scrapped parts whose origin status fell outside those 3. Built as a genuinely separate,
unconstrained query instead, exactly as the ticket's own dev notes recommended.

**What got built:**
- **Backend** (`inventory.service.ts`):
  - `getPartsResolutionBuckets` — new independent query: `COUNT(*)`/`SUM(poUnitPrice)` WHERE
    `resolutionStatus = 'Scrapped'`, no status/window filter at all. Return type gained
    `scrapped: { count, value }`.
  - `getPartsInventoryDrilldown`'s `RESOLUTION_STAGE` case — new `SCRAPPED` branch inserted
    **before** the shared `flaggedStatuses` filter is applied (early `break`), so the drill-down
    list matches the tile's count exactly. The existing 4-stage status/window logic is untouched.
  - Both `resolutionStage` type unions (drilldown params + bucket type) widened with `"SCRAPPED"`.
  - No migration — `resolutionStatus`'s `Scrapped` enum value already existed.
- **Frontend:**
  - `types/inventory.ts` + drilldown panel's `resolutionStage` type — both widened to match.
  - `overview_tab.tsx` — new **standalone row** (user's explicit call, not inside "Parts for
    Resale") below "Return Window Unknown", **solid** stone/muted styling (`#F5F1EE`/`#57534E`) —
    deliberately not dashed, since dashed already signals "data gap" (Return Window Unknown) and
    Scrapped is a closed/final outcome, not a gap. 5th segment added to the proportional bar/legend
    (`#57534E`, matching the Scrapped badge color already used elsewhere in the app —
    `part_note_panel.tsx`'s `BADGE_STYLE.Scrapped`).

**Decisions confirmed with the user before building:**
- **Value shown = `poUnitPrice`** (same field all 4 existing buckets already use) — framed as
  "money permanently written off," not a resale/listing price. No scrap-value/write-off-amount
  field exists anywhere in the schema (confirmed back in AG-260/263 — client said no such column
  was needed).
- **Visual placement = standalone row**, not a 3rd card inside "Parts for Resale" — the ticket's
  own Notes for Designer flagged this as genuinely unresolved ("Scrapped is a closed/final outcome
  rather than pending — confirm whether it belongs in the same visual framing"); user picked
  standalone, matching my recommendation.

**Related:** [[issue-AG-260-resold-resolution-path]] (where Scrapped itself, and the
`TERMINAL_RESOLUTION_STATUSES` exclusion this ticket had to work around, were first built).

---

## HISTORY

- 2026-09-10: User opened with just "new ticket: AG-285" and no details — first time this session a
  ticket wasn't accompanied by a description/screenshot. Atlassian Rovo MCP was disconnected at
  that point; told the user I couldn't fetch it and asked them to paste the details.
- 2026-09-10: User ran `/ready` instead; noticed mid-turn the Atlassian connector had reconnected
  (new deferred tools appeared). Reported this in the `/ready` output rather than silently fetching.
- 2026-09-10: User repeated "new ticket: AG-285" — fetched it directly via `getJiraIssue` this time
  (first successful non-interactive Jira fetch in this exact session; connector had needed
  reconnecting via `/mcp` in an interactive terminal previously). Ticket had unusually thorough
  developer notes already written (line numbers, exact query names) — re-verified every one against
  current code rather than trusting them, since 3 tickets' worth of edits (AG-261/270/284) had
  shifted line numbers since those notes were written. Found the real complication (Scrapped's
  any-origin-status behavior) independently by reading `markTaskPartAsScrapped`/
  `markTaskPartAsListed` directly. Explained understanding + plan, flagged the ticket's own
  unresolved Designer question (standalone vs. grouped placement) rather than picking one silently,
  and asked permission.
- 2026-09-10: User confirmed standalone row, then asked directly which price field would be shown
  before allowing the build to proceed — answered (`poUnitPrice`, same field as the other 4
  buckets, framed as "written off") before implementing, per the user's explicit sequencing ask.
  Built both repos. `tsc` clean both, `next lint` clean on all 3 changed frontend files.
- 2026-09-10: User asked directly whether AG-285 was tracked as the active ticket — it wasn't yet
  (same oversight as AG-261). Created this file, added to `MEMORY.md`'s active index.
