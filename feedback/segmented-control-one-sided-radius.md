---
name: segmented-control-one-sided-radius
description: When a Figma/design export shows one segment with only right-side border-radius and another with only left-side, that's a seamless multi-segment bar, not a padded floating-pill toggle
metadata:
  type: feedback
---

When given exact CSS/design tokens for a 2-option toggle where one segment has
`border-top-right-radius`/`border-bottom-right-radius` only, and the other has
`border-top-left-radius`/`border-bottom-left-radius` only (no radius on the touching inner edge),
that describes a **seamless bar of adjacent, equal-height segments with no gap between them** —
rounded only at the two outer ends of the whole control. It is NOT a small "active" pill floating
inside a padded outer track (the more common toggle pattern, and the default guess when styling
from a screenshot alone without the exact tokens).

**Why:** Built AG-278's People-tab role toggle and Alert Checks tab bar from screenshots alone
first — guessed a padded outer pill (`bg-[#F2F2F6] rounded-full p-1`) with a smaller floating active
segment inside, plus a self-picked near-black active color. User then supplied the real Figma
tokens: exact background colors per state, AND the one-sided radius per segment described above.
Rebuilding correctly meant removing the padded wrapper entirely and giving each segment its own
full-height background, with only the first segment in the row getting `rounded-l-full` and only
the last getting `rounded-r-full` — middle segments (when there are more than 2, e.g. 3 or 4 tabs)
get no rounding at all.

**How to apply:** When exact CSS is given (not just a screenshot) and a segment's radius is
one-sided, build a seamless bar: no outer padding/background wrapper, each segment fills its own
slot edge-to-edge, and only the first/last segments in the row get outer rounding
(`rounded-l-full`/`rounded-r-full`). Generalizes cleanly to any number of segments, not just the 2
shown in the example.
