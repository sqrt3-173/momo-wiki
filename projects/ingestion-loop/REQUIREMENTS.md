# Requirements: Ingestion + Autonomous Loop

**Defined:** 2026-07-03
**Core Value:** Eli dumps freely and sleeps; the machine turns the pile into filed knowledge, advanced projects, and decisions worth making — with receipts.

> Source: [[../ingestion-loop-design]] (binding design session). Project: [[PROJECT]] · Roadmap: [[ROADMAP]].

## v1 Requirements

### Ingestion

- [ ] **ING-01**: Inbox stores raw dumps as immutable typed items (text, transcript, image, video-ref, file) — never edited, never deleted.
- [ ] **ING-02**: Eli captures by messaging MOMO on Discord (a note/idea/dump) — item lands with an instant ✓ acknowledgement.
- [ ] **ING-03**: Ingestion decomposes each item into atoms (smallest self-contained thoughts), each atom typed, each carrying provenance to its source item; items link to their atoms.
- [ ] **ING-04**: Per-type extraction handlers: text/transcript read for atoms; images viewed and described into the record; video references fetched/distilled via research agent. New type = new handler, same atom model.

### Triage (reasoning, not rules)

- [ ] **TRI-01**: Every atom is evaluated by a reasoning agent against the goals layer: does it matter, how much (magnitude judgment), highest-leverage response — with the reasoning logged.
- [ ] **TRI-02**: Atom fates: knowledge → wiki (filed by subject, linked); project idea → pipeline at Eli's gate; task → existing project with blast-radius lane (plain task vs planned change); seed → parked with review date.
- [ ] **TRI-03**: A stated goal is drilled, not filed: decomposed into drivers, constraint located from known facts, gaps researched, strategy brief + proposed roadmap changes delivered to Eli.
- [ ] **TRI-04**: Genuine ambiguity produces sharp numbered questions to Eli, never guesses.

### Goals layer

- [ ] **GOAL-01**: A per-business goals page in the wiki (direction, constraints, economics) that triage reads and Eli edits; a goal stated once on Discord updates it.

### Seeds

- [ ] **SEED-01**: Seeds carry review dates and full provenance.
- [ ] **SEED-02**: Weekly synthesis review clusters all live seeds + recent knowledge atoms; critical-mass clusters get pre-work (research, fit-vs-existing-systems, mini-brief); output is a shaped numbered report to Eli (start / merge-park / kill), incl. change-sets against existing projects.
- [ ] **SEED-03**: Every retirement carries a written reason; nothing disappears silently.

### The heartbeat loop

- [ ] **LOOP-01**: A scheduled tick (launchd/cron, ~30 min) runs a shell pre-check against the DB (ready plans? untriaged items? seeds due?) and exits near-free when idle.
- [ ] **LOOP-02**: When work exists, the tick wakes a fresh Claude work session that claims one unit (plan/triage/review), executes it under full pipeline discipline (guard, telemetry, digest), and closes cleanly.
- [ ] **LOOP-03**: "pause" / "resume" from Eli on Discord stops/starts the loop (engine-state flag the tick respects).
- [ ] **LOOP-04**: Every tick session records telemetry (start_run/log_event/finish_run) and lands its outcomes in the digest.

### Visibility

- [ ] **VIS-01**: Dashboard inbox panel: items → their atoms → each atom's fate + reasoning, with provenance links.
- [ ] **VIS-02**: Home status strip shows loop state: last tick, next expected, paused/active.

## v2 Requirements

- **ING-05**: Automatic speech-to-text for raw voice memos.
- **LOOP-05**: Parallel executor fan-out (blocked on Eli's guard decision).

## Out of Scope

| Feature | Reason |
|---------|--------|
| Auto-approving spend/publish/install/delete/contact | Authority stays with Eli — permanent |
| Rules/pattern engine for triage | Eli's explicit call: reasoning from goals, not pre-imagined patterns |

## Traceability

| Requirement | Phase | Requirement | Phase |
|-------------|-------|-------------|-------|
| ING-01..04 | Phase 1 | SEED-01..03 | Phase 3 |
| GOAL-01, TRI-01..04 | Phase 2 | LOOP-01..04 | Phase 4 |
| VIS-01..02 | Phase 5 | | |

**Coverage:** 18/18 mapped, 0 unmapped ✓

---
*Requirements defined: 2026-07-03 (from the Discord design session)*
