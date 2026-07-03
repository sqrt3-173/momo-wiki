# Roadmap: Ingestion + Autonomous Loop

> Rendered from `momo_work` (Ingestion + Autonomous Loop v1.0) — do not hand-edit; the DB is the source of truth.

## Overview

Always-on layer of the work engine: immutable typed inbox, atom extraction with provenance, goals-as-context reasoning triage, seeds with weekly synthesis reviews, scheduled heartbeat tick with cheap pre-check, Discord pause/resume, dashboard inbox panel + loop status.

## Phases

- [ ] **Phase 1: Capture & Decomposition** — Anything Eli dumps on Discord lands instantly as an immutable typed inbox item and is decomposed into typed atoms with two-way provenance
- [ ] **Phase 2: Goals Layer & Reasoning Triage** — Every atom receives a reasoned fate judged against the live goals layer — knowledge filed, tasks routed, ideas gated, goals drilled — with the reasoning logged
- [ ] **Phase 3: Seeds & Weekly Synthesis** — Parked ideas compound instead of rotting — a weekly synthesis pass turns seed clusters into shaped, pre-worked decisions with written reasons for every retirement
- [ ] **Phase 4: Heartbeat Loop** — The engine works unattended — a near-free scheduled tick finds real work, wakes a governed fresh session to execute one unit, and lands its outcomes with receipts
- [ ] **Phase 5: Visibility** — Eli sees the whole loop at a glance — items to atoms to fates with reasoning on the dashboard, and loop health on the home strip

## Phase Details

### Phase 1: Capture & Decomposition
**Goal**: Anything Eli dumps on Discord lands instantly as an immutable typed inbox item and is decomposed into typed atoms with two-way provenance
**Depends on**: Nothing (first phase)
**Requirements**: ING-01, ING-02, ING-03, ING-04
**Success Criteria** (what must be TRUE):
  1. Eli messages MOMO a note/idea/dump on Discord and gets an instant ✓ acknowledgement; the dump is stored as an immutable typed inbox item (append-only — rows can't be edited or deleted)
  2. Each item is decomposed into atoms (smallest self-contained thoughts), each atom typed and traceable to its source item, and the item lists its atoms
  3. An image dump is viewed and described into the record; a video reference is fetched/distilled via the research agent — each type through its own extraction handler on the same atom model
  4. A past item can be re-read with its full atom trail intact (originals untouched)
**Plans**: 3 plans

Plans:
- [x] 01-01: Immutable inbox substrate (010-ingestion.sql + DB proofs)
- [x] 01-02: Extraction protocol: extractor agent prompt + capture runbook
- [x] 01-03: End-to-end proof: real capture, per-type extraction, provenance + immutability live

### Phase 2: Goals Layer & Reasoning Triage
**Goal**: Every atom receives a reasoned fate judged against the live goals layer — knowledge filed, tasks routed, ideas gated, goals drilled — with the reasoning logged
**Depends on**: Phase 1
**Requirements**: GOAL-01, TRI-01, TRI-02, TRI-03, TRI-04
**Success Criteria** (what must be TRUE):
  1. Each business has a goals page in the wiki (direction, constraints, economics) that Eli can edit directly; a goal stated once on Discord updates it
  2. Every atom is evaluated by a reasoning agent against the goals layer — relevance, magnitude, highest-leverage response — with the reasoning logged and readable
  3. Fates land for real: a knowledge atom is filed and linked in the wiki, a task atom reaches its project in the right blast-radius lane, a project idea arrives at Eli's gate, a seed atom is parked
  4. A stated goal is drilled, not filed: decomposed into drivers, binding constraint located, gaps researched, strategy brief + proposed roadmap changes delivered to Eli
  5. A genuinely ambiguous atom produces sharp numbered questions to Eli on Discord — never a guessed fate
**Plans**: 4 plans

Plans:
- [x] 02-01: Goals layer: per-business goals pages + wiki index
- [x] 02-02: Triage agent + fate-execution runbook + reasoning-invariant test
- [x] 02-03: Goal-drill flow: driller agent + drill runbook section
- [x] 02-04: Live end-to-end triage proof over the full pending set at run time

### Phase 3: Seeds & Weekly Synthesis
**Goal**: Parked ideas compound instead of rotting — a weekly synthesis pass turns seed clusters into shaped, pre-worked decisions with written reasons for every retirement
**Depends on**: Phase 2
**Requirements**: SEED-01, SEED-02, SEED-03
**Success Criteria** (what must be TRUE):
  1. Every parked seed carries a review date and full provenance back to its source dump
  2. The weekly review runs as one synthesis pass — clustering all live seeds plus recent knowledge atoms, never a per-seed walk
  3. Critical-mass clusters arrive with pre-work attached: research, fit-vs-existing-systems, mini-brief
  4. Eli receives a shaped numbered report (start / merge-park / kill) including change-sets against existing projects
  5. Every retirement carries a written reason — nothing disappears silently
**Plans**: TBD (planned during plan-phase)

### Phase 4: Heartbeat Loop
**Goal**: The engine works unattended — a near-free scheduled tick finds real work, wakes a governed fresh session to execute one unit, and lands its outcomes with receipts
**Depends on**: Phase 2, Phase 3
**Requirements**: LOOP-01, LOOP-02, LOOP-03, LOOP-04
**Success Criteria** (what must be TRUE):
  1. An idle tick runs its shell pre-check against the DB (ready plans? untriaged items? seeds due?) and exits near-free — no Claude session started
  2. When work exists, the tick wakes a fresh claude -p session that claims exactly one unit (plan/triage/review), executes it under full pipeline discipline (guard, telemetry, digest), and closes cleanly
  3. Eli types "pause" on Discord and ticks stop doing work; "resume" restarts them — the engine-state flag is respected by the pre-check
  4. Every tick session shows up in telemetry (start_run/log_event/finish_run) and its outcomes land in the digest
  5. Overnight proof: a dump sent before bed is triaged and filed by morning with no one in a conversation
**Plans**: TBD (planned during plan-phase)

### Phase 5: Visibility
**Goal**: Eli sees the whole loop at a glance — items to atoms to fates with reasoning on the dashboard, and loop health on the home strip
**Depends on**: Phase 2, Phase 4
**Requirements**: VIS-01, VIS-02
**Success Criteria** (what must be TRUE):
  1. The dashboard inbox panel drills item → its atoms → each atom's fate + logged reasoning, with provenance links both directions
  2. The home status strip shows live loop state: last tick, next expected, paused/active
  3. A fresh Discord dump appears in the panel with its atoms and fates after triage — the panel reflects live engine data, not a snapshot
**Plans**: TBD (planned during plan-phase)

## Progress

| Phase | Plans Complete | Stage | Status |
|-------|----------------|-------|--------|
| 1. Capture & Decomposition | 3/3 | verified | In progress |
| 2. Goals Layer & Reasoning Triage | 4/4 | verified | In progress |
| 3. Seeds & Weekly Synthesis | 0/0 | todo | Not started |
| 4. Heartbeat Loop | 0/0 | todo | Not started |
| 5. Visibility | 0/0 | todo | Not started |

