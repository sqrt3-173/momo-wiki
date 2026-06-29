---
name: website-audit
description: Legal + security + quality audit of a target business's website, producing an actionable report and a "we could build better" angle.
---

# website-audit

The audit MOMO runs on a BD target's existing site to find the gaps a better build would close.
Feeds the [[bd-research-sweep]] pipeline.

## When to use
- Auditing a prospect's current website as part of BD (opportunity scoring, "draft something better").
- Eli asks for a technical/legal/design critique of any site.

## Inputs
- Target name + URL. Cross-check it's the right entity (generic names mislead).
- Web access (search + fetch). Snippets-only for gated sites (don't stall on 403s).

## Method — three lenses
1. **Legal**
   - APP-5 collection notice present on every form that collects personal info?
   - Privacy policy present + adequate (states what's collected, why, how stored, contact)?
   - Terms of service present?
   - Health sector: any extra obligations (health records handling) flagged?
2. **Security**
   - Identify the stack. Flag fragile/high-maintenance ones (e.g. WordPress + plugin sprawl).
   - Exposed staging clones, leaked admin paths, obvious version disclosure, missing HTTPS.
3. **Quality**
   - Design/usability critique (clarity, trust signals, mobile, conversion path).
   - Performance / page-rank scores (load speed, Core Web Vitals signals).
   - Homepage report: first impression, what's missing, what converts.

## Output contract
A tight report: per-lens findings with severity, then a ranked **"better build" angle** — what we'd
fix and why it matters commercially. Honest confidence; absence noted as data ("no privacy policy
found" ≠ "didn't check"). Persist key findings to the CRM/graph (WebPresence + notes) and capture via [[wiki-capture]].

## Gotchas
- **Cap web fetches** (~6–8) and skip slow/blocking sites — uncapped fetches stall the watchdog.
- **Don't fetch gated platforms directly** (LinkedIn/Glassdoor/etc.) — use search snippets.
- **Disambiguate first** — confirm it's the right business before auditing the wrong site.
- Distinguish template-grade (easy to beat/displace) vs genuinely bespoke builds.
