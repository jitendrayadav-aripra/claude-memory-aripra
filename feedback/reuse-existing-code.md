---
name: reuse-existing-code
description: Always search for an existing method/helper before writing new code; reuse over reinvent
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 85508222-fb04-4af4-b07c-2b4d62f2d874
---

Before writing or rewriting any code, **search the codebase for an existing method/helper that already
does it** and reuse it. Don't author a new function when one exists.

**Why:** the user repeatedly catches reinvented logic and points to the existing helper — it keeps the
code consistent, avoids divergent/buggy duplicates, and matches how the app already behaves. Examples this
project: reused `getAllPendingShares` (the app's existing pending-shares count behind the badge / bg
notification) instead of adding a new `pendingShareCount` to SharingModel (#618); reused
`removeFilesAtImagesSubdirectory:` / `MediaItemModel deleteMedia` / `showResetDialog:` rather than new ones.
Also earlier: "Why a new sh file — why can't we do with existing code?" (rejected a shell-script reimpl of
a fastlane lane; fixed the env instead).

**How to apply:**
- grep for the capability (method names, similar call sites, sibling view controllers) BEFORE writing — and
  before proposing an approach in a plan/ticket, so estimates assume reuse.
- When porting from LumaBooth → LumaShare, first check what LumaShare ALREADY has (it often has the field/
  method under a different name/architecture — e.g. `PreferenceObject` vs `LBEventPreferences`).
- Prefer extending/calling existing helpers over new ones; match existing patterns (NSCoding fields,
  `checkBoxAction:`, dialog helpers) rather than introducing parallel mechanisms.
- If a new method is genuinely needed, say why the existing ones don't fit.

Related: [[github-reply-tone]] (verify facts against the codebase before asserting them).
