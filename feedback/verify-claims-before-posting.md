---
name: verify-claims-before-posting
description: "Before posting any finding to GitLab, verify every claim against the actual pasted logs/diff line-by-line — never carry forward a narrative from an earlier test"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 949bc40a-0319-4fca-a1a5-306f086b798d
---

Before posting (or committing) any analysis, verify EACH factual claim against the actual evidence in
front of me — the pasted logs, the `git diff`, the real timestamps — line by line. Do not restate a
narrative from an earlier test, an earlier session, or memory as if it were observed in the current data.

**Why:** this caught me on #1930 — I posted that "a 429 came through without signing out, then a 401
fired the sign-out," implying a tolerate-then-escalate sequence. The actual logs showed the storm was
401-driven and the lone 429 was on a different call in the SAME second as the sign-out — never a separate
ridden-through event. The wording overstated what the logs proved and had to be corrected on GitLab. A
plausible-sounding but unverified claim erodes trust with Mike/Marty and creates rework.

**How to apply:**
- Pull the specific lines (`grep`/python over the pasted log or transcript) and quote them; don't
  paraphrase or summarize from memory.
- Check the TEMPORAL order — same-second events are not a sequence; one error type is not "then" another.
- Separate what the evidence PROVES from what it's merely CONSISTENT with; phrase accordingly
  ("the 429 did not itself trigger sign-out" vs "we rode through a 429").
- If a claim isn't backed by a line I can point to, soften it or drop it ("still replicating") rather
  than assert it — see [[support-ticket-comment-structure]].
- **Same rule applies BEFORE drafting a cross-stack plan, not just before posting.** For any ticket whose
  design depends on how data is stored/synced on another platform (Windows ⇄ iOS ⇄ Firebase/dashboard),
  read that platform's actual storage/sync code FIRST — don't assume one screen's model carries to another.
  Caught me on #1953 (2026-06-25): I drafted the iOS plan assuming sharing/capture were inline like the
  start screen; reading the Windows code proved sharing is an asset-file ref and capture isn't synced at all,
  forcing a full re-plan. Verify the ground truth, then plan — a plan built on an unchecked assumption is the
  same wasted rework as an unverified posted claim, just more expensive.

Pairs with [[github-reply-tone]] (confirm before posting) and [[read-google-cloud-logs]].
