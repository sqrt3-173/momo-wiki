---
name: council-review
description: Multi-agent adversarial review for high-stakes calls — generate, then independently verify/refute before committing. Claude-only now; multi-model when the council is funded.
---

# council-review

The "be confident" pattern: for decisions where being wrong is expensive, don't trust a single pass.
Generate, then have independent reviewers try to *refute* before you commit.

## When to use
- High-stakes calls: architecture decisions, audits, ambiguous BD inferences, anything load-bearing.
- **Not** routine/mechanical work — it's wasteful overkill there.

## Inputs
- The claim/finding/design to test. Web+read for verifiers as needed.

## Method
1. **Generate** the finding or N candidate approaches.
2. **Verify adversarially** — spawn independent reviewers, each prompted to *refute* (default to "refuted if uncertain"). For findings that can fail multiple ways, give each verifier a distinct lens (correctness / security / reproducibility / commercial).
3. **Decide by majority** — kill the finding if ≥majority refute; keep what survives.
4. **Synthesise** — for design choices, score candidates with independent judges and build from the winner, grafting the best ideas from runners-up.
5. **Report** what survived + what was killed + what could NOT be verified (no fabricated confidence).

## Two tiers
- **Now (Claude-only):** Opus + Sonnet/Haiku in different roles via the Agent tool / Workflow orchestration. Already gets most of the "multiple minds" benefit. (This is how the bariatric findings were verified.)
- **Later (multi-model):** add external models (GPT-5, Gemini) via an OpenRouter key (Eli funds — MOMO is hard-blocked from spending). Cross-model diversity catches blind spots one model misses, because different models fail differently. Wire as a fan-out: same question → Claude + 1–2 outside models → synthesise + adversarial-verify.

## Output contract
- A verdict with explicit survivors, rejections, and unverifiable gaps. Confidence honest.

## Gotchas
- **Diversity > redundancy** — distinct lenses beat N identical refuters for multi-failure-mode claims.
- **Scope to stakes** — match council depth to how expensive being wrong is.
- **Cross-model costs money** — keep it for the genuinely hard calls; don't burn API spend on routine review.
