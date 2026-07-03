<!-- Goals page: Eli edits this directly in Obsidian; MOMO's triage reasons from it.
     A goal stated once on Discord ("goal: ...") updates this page.
     Provenance convention: machine appends carry a comment naming atom id, item id, date. -->

# Goals — MOMO Engine (the meta-business)

## Direction
<!-- source: wiki/projects/ingestion-loop-design.md "The system in one line"; ingestion-loop PROJECT.md Core Value -->
An always-on system that works through Eli's projects with agents thinking critically about
what to do next: Eli dumps freely and sleeps; the machine decomposes, reasons against his
businesses' goals, files knowledge, advances projects, parks and resurfaces seeds — receipts
for everything. The engine exists to multiply Eli, not to be admired.

**In Eli's own words (the sentence that governs priority calls):** "a smart orchestration of
agents to work through projects, keep me in the loop, but most importantly be quite
autonomous, and always working on something, and having critical thinking for giving
suggestions that expand beyond current idea set for me to approve and breakdown even more."
The weighting is his: autonomy + always-working ranks first; expansive critical suggestions
route to his gate for approval and further breakdown. (Auto-watch/second-brain explicitly
NOT fundamental — parked long, same message.)
<!-- atom 27 · item 9 · 2026-07-03 -->


## Constraints
<!-- source: ingestion-loop PROJECT.md "Constraints" + D-05; soul.md autonomy rules -->
- Substrate: momo_work Postgres + wiki + Discord + guard + telemetry + dashboard.
- Originals immutable; triage is reasoning against goals, never pattern rules.
- Idle must cost ~nothing (cheap-check-first heartbeat).
- Action boundary is authority, not intelligence: research/quantify/draft autonomous;
  contact/spend/publish/install/delete NEVER autonomous; guard governs every session.

## Economics (what's known)
<!-- source: wiki/ops/model-cost-reference.md; DL-17 (work-dashboard DECISIONS) -->
- Cost discipline exists (blended token rates, telemetry cost_usd at write time) but the
  session model (Fable 5) has no recorded rate yet — all runs currently unpriced (DL-17).
- **Engine run-rate budget: Not yet recorded — Eli to state if he wants a ceiling.**

> Strategy brief: [[momo-engine-strategy-2026-07-03]] — binding constraint located
> (session-bound engine, gated by cost visibility); 3 moves proposed at Eli's gate.

## Current priorities
<!-- source: momo_work roadmap rows (ingestion-loop phases 1-5); work-dashboard DECISIONS DL-19 -->
- Ingestion-loop build: phase 1 (capture) verified; phase 2 (goals + triage) executing;
  then seeds, heartbeat, visibility.
- Dashboard v1 complete; backlog: design-council round 2 items, Fable rates backfill.
- Standing debt: parallel-executor guard decision (Eli's call, when he's ready).
