# Design fidelity — no shortcuts (Eli feedback, 2026-07-04)

**When building to a design reference, match it exactly — wire the real assets, port the
animations, replicate the exact layout mechanism. Never approximate or fall back to
placeholders silently.**

On the NV Health v4 landing page, Eli reviewed and caught several corners I'd cut:
- Used initials/text placeholders instead of the real team photos + procedure diagram SVGs —
  which were all present in the reference (I'd looked in the wrong folder: `uploads/`, not
  `ui_kits/website-fluid/assets/`).
- Dropped the pulse/ripple + drift animations entirely.
- Simplified the 5-stage timeline into a wrong `flex-wrap` layout ("containers wrap/direction
  is off") instead of the v4's equal-column flex row + dividers + between-stages marker + count pills.
- Used an auto-fit 3+1 credential grid instead of the intended 2×2.

## Why this matters
Eli's standard (see [[../../soul]]) is best-or-nothing — "architected so well it sings" — and he
reviews design work closely. An approximation that "looks close" reads as not caring. Cutting a
corner silently and hoping it passes is precisely the trust-eroding failure mode the soul file warns
about. He'd rather I take longer and match the reference than ship a shortcut. His words: "just try
harder… keep holding me to the design."

## How to apply (checklist for any design-replication build)
1. **Find the real assets before ANY fallback.** Search the whole design-reference tree
   (`find <ref> -iname "*.svg" -o -iname "*.png" …`) for every referenced asset. Only use a
   placeholder if it genuinely doesn't exist — and then SAY SO explicitly, never substitute silently.
2. **Port animations/interactions, not just static layout.** Pull the reference's actual `<style>`
   keyframes + classes and replicate them (with a `prefers-reduced-motion` fallback). Pulses, hovers,
   transitions, drift — all count.
3. **Replicate the exact layout mechanism**, not an approximation. Read the reference CSS. The v4
   timeline was an equal-column flex row with hairline dividers + a zero-width marker + `@container`
   stacking — NOT flex-wrap. Match the mechanism.
4. **Assume layout specifics are intentional** (2 cols vs 3, exact overlaps, gradient stops). Eli notices.
5. Related build patterns live in [[../projects/nv-health-website]] (layout-in-CSS, adherence gate, BookCta).
