---
name: entity-enrichment
description: Deep-enrich graph entities (companies, staff, agencies, referrers) via concurrency-capped research agent waves → harvest → idempotent write.
---

# entity-enrichment

The agent-wave mechanics behind [[bd-research-sweep]]. Battle-tested on 458+ allied-health staff +
104 companies. Lives as code in `projects/bd-crm/enrich/`.

## When to use
- Bulk deep-profiling of non-surgeon graph entities (or any entity set) needing web research + structured persistence.

## Inputs
- Un-enriched entities in the graph (DB `enrichedAt IS NULL` is the source of truth).
- The research brief / output contract: `enrich/entity_schema.md`.

## Method (pipeline)
1. **Launch a wave** of ≤6 research agents (8 OK, 10 stalls), each with the brief from `enrich/entity_schema.md`, **web+read only**.
2. **Harvest** final JSON from the agent transcripts (out-of-context): `python3 enrich/harvest_entity.py <tasks_dir> <out.json> <taskids>`.
3. **Persist** idempotently: `bun run enrich/write_entity.ts <file> <YYYY-MM-DD>` — dossier→notes (prefixed `ENRICHED <date>`), fees→PricePoint, socials/reviews/edges→graph. Delete-then-insert, so re-running is safe.
4. **Bank + relaunch** the next wave on completion (completion-driven). A ~900s `ScheduleWakeup` is only a failsafe.
5. Repeat until the un-enriched count hits 0.

## Output contract
- Entities updated with structured dossier + facts; fees as queryable PricePoint rows; socials/reviews/edges in the graph. Idempotent.

## Gotchas
- **Concurrency ceiling ≈6 agents** per instance (10 stalled historically). Across two instances (MOMO+NUNU) keep ≈12 total — see [[../../ops/multi-instance]].
- **Cap each agent's web fetches** (~6–14) and skip slow/blocking sites — uncapped agents stalled the 600s watchdog (one zombie ran ~1.5h).
- **Never directly fetch** LinkedIn/Glassdoor/Yelp/HealthShare — they gate + stall. Snippets only.
- **Don't Read/tail agent `.output` files via shell** — overflows context. Use the harvest script.
- **DB is the source of truth** — harvest always re-filters to `enrichedAt IS NULL`; safe to crash + resume.
- Agents return JSON/dossier even when data is unknown; honest confidence, never fabricate.
