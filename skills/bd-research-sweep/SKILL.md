---
name: bd-research-sweep
description: End-to-end BD pipeline — deep-research targets, enrich the CRM graph, score the opportunity, audit their site, draft a better build, report to Eli.
---

# bd-research-sweep

MOMO's core BD motion. Turns a raw target list into scored, enriched, actionable opportunities Eli
can run discovery + sales on. Proven on the Australian bariatric market (see [[../../projects/bariatric-intelligence]]).

## When to use
- New BD vertical or target list (e.g. GP practices across SEQ, bariatric surgeons).
- Refreshing/deepening intelligence on an existing set.

## Inputs
- Target definition (vertical + geography) or an explicit list.
- The CRM graph (Postgres, `bd-crm`). Web access.

## Method
1. **Scope + seed.** Define the target universe; seed the graph with nodes (practices, practitioners, hospitals).
2. **Deep research** via agent waves — use [[entity-enrichment]] for the mechanics (≤6 agents/wave, fetch caps, snippets for gated sites).
3. **Enrich the CRM.** Persist structured intel: practice, practitioners, fees (PricePoint), socials, reviews, edges, key details. Honest confidence; label inferences.
4. **Score the opportunity.** Rank by fit + need signals (fragile site, weak digital footprint, high ambition, under-marketed strengths). The best targets: real substance, poor current packaging.
5. **Audit the site** via [[website-audit]] for each qualified target.
6. **Draft something better** — concrete "here's the improved build" angle.
7. **Report to Eli** — phone-first Discord: lead with the answer, ranked shortlist, the wedge for each. He runs the human side (discovery + sales).
8. **Capture** learnings + methodology updates via [[wiki-capture]].

## Output contract
- Enriched graph (queryable, comparable — e.g. per-practitioner fees as structured rows).
- A scored shortlist + per-target BD wedge, delivered to Eli.
- Wiki note documenting the sweep, findings, and any premise corrections.

## Gotchas
- **Don't fabricate.** Structural inference is fine *labelled as such*; never present a guess as a confirmed fact. (E.g. "politics" = structural inference, not invented feuds.)
- **Dedup the graph** — near-duplicate nodes get merge-flagged; act on the canonical node.
- **Correct premises out loud** — when research disproves a brief's assumption, surface it (deceased surgeon, mis-attributed paper, wrong partnership).
- **Absence is data** — record "no published fee" explicitly so we never confuse "didn't look" with "they don't publish".
