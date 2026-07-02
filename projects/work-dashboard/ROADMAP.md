# Roadmap: Work-engine dashboard

> Rendered from `momo_work` (Work-engine Dashboard v1 (Mission Control) v1.0) — do not hand-edit; the DB is the source of truth.

## Overview

Local, read-only mission-control web app for the work engine: projects/phases/plans progress, live agent activity, Eli's queue + what it blocks, overnight digest, usage/cost telemetry, guard audit, host health. Next.js + shadcn on the mini, reads momo_work directly.

## Phases

- [ ] **Phase 1: Projects at a Glance** — Eli opens one local URL and immediately sees every project the engine is running, with live status and progress, straight from the work DB
- [ ] **Phase 2: Project Drill-down & Wave Lanes** — Eli can descend from any project into its full structure and see execution order at a glance without a Gantt chart
- [ ] **Phase 3: Blocked-on-You Queue** — Eli sees exactly what is waiting on him and what each item is holding up, ranked so the most-blocking decision is first
- [ ] **Phase 4: Telemetry & Live Agents** — The engine records what agents do as they do it, and Eli watches agents working right now — the recording layer ships with its first live view proving it
- [ ] **Phase 5: History, Spend & Overnight Digest** — Eli wakes up, opens the dashboard, and knows what happened while he slept — what ran, what it did step by step, and what it cost
- [ ] **Phase 6: Guard & Host Health** — Eli sees what the guard decided on his behalf and whether the machine itself is healthy — the ops picture that closes out mission control

## Phase Details

### Phase 1: Projects at a Glance
**Goal**: Eli opens one local URL and immediately sees every project the engine is running, with live status and progress, straight from the work DB
**Depends on**: Nothing (first phase)
**Requirements**: VIEW-01
**Success Criteria** (what must be TRUE):
  1. Eli opens the dashboard on his MacBook over the home network and sees all projects listed
  2. Each project shows status and done/doing/todo progress rollups that match what's in momo_work
  3. Data on screen updates from the live DB (refresh reflects real state, no stale seed data)
  4. The app connects with read-only credentials — attempting any write path fails by construction
**Plans**: 4 plans

Plans:
- [x] 01-01: Scaffold: Next 16 + Tailwind 4 + shadcn + test harness (bun-only)
- [x] 01-02: Checkpoint: Node 24 LTS runtime install (Eli-gated)
- [x] 01-03: Read-only data layer: dashboard_ro role + typed queries, test-first
- [x] 01-04: Projects-at-a-glance UI: dynamic SSR grid + auto-refresh, LAN-verified

### Phase 2: Project Drill-down & Wave Lanes
**Goal**: Eli can descend from any project into its full structure and see execution order at a glance without a Gantt chart
**Depends on**: Phase 1
**Requirements**: VIEW-02, VIEW-03
**Success Criteria** (what must be TRUE):
  1. Eli clicks a project and sees milestone → phases → plans → tasks with stage and status at each level
  2. Requirements coverage is visible per phase (which REQ-IDs it delivers, mapped or orphaned)
  3. A phase's plans render as wave-lane columns in execution order, colored by status
  4. Each plan shows its blocked-by list so Eli can trace why a lane hasn't started
**Plans**: 3 plans

Plans:
- [x] 02-01: Project drill-down route + hierarchy slice
- [x] 02-02: Wave lanes with blocked-by tracing
- [x] 02-03: Requirements coverage badges per phase

### Phase 3: Blocked-on-You Queue
**Goal**: Eli sees exactly what is waiting on him and what each item is holding up, ranked so the most-blocking decision is first
**Depends on**: Phase 2
**Requirements**: QUEUE-01, QUEUE-02
**Success Criteria** (what must be TRUE):
  1. Eli sees all his open gates and human tasks in one queue view
  2. Queue items are ranked by how much downstream work each one blocks
  3. Expanding a queue item shows the specific downstream plans/tasks it is holding up
  4. Clearing a gate in the engine removes it from the queue on next load
**Plans**: 2 plans

Plans:
- [x] 03-01: Queue data seam + pure view-builder (test-first)
- [x] 03-02: /queue route: ranked blocked-on-you view with expansion

### Phase 4: Telemetry & Live Agents
**Goal**: The engine records what agents do as they do it, and Eli watches agents working right now — the recording layer ships with its first live view proving it
**Depends on**: Phase 1
**Requirements**: USE-01, AGENT-01
**Success Criteria** (what must be TRUE):
  1. Every agent run writes telemetry to momo_work: run start/end, host, plan/task, model tier, tokens, cost, and breadcrumb events
  2. Eli sees currently-running agents with host, project, plan/task, and started-at
  3. An agent that starts while the dashboard is open appears in the live view; one that finishes drops off
  4. A run's recorded tokens/cost reconcile with what the run actually consumed
**Plans**: 3 plans

Plans:
- [x] 04-01: Telemetry schema: agent_runs/agent_events, writer trio, live/recent views
- [x] 04-02: /agents route: live section + recent-runs strip
- [x] 04-03: Proof-of-life: orchestrator records its own session; tokens/cost reconcile

### Phase 5: History, Spend & Overnight Digest
**Goal**: Eli wakes up, opens the dashboard, and knows what happened while he slept — what ran, what it did step by step, and what it cost
**Depends on**: Phase 4
**Requirements**: AGENT-02, AGENT-03, USE-02, USE-03, DIG-01
**Success Criteria** (what must be TRUE):
  1. Eli opens a running agent and sees a live breadcrumb feed of its recent actions
  2. Eli sees past runs with outcome, duration, and cost per run
  3. Eli sees spend per project over time, and totals agree with the per-run figures
  4. Eli sees a chronological timeline of engine events (plans completed, gates cleared, commits, DMs sent, spend)
  5. Eli filters the timeline to a time window ('overnight') and sees only that window's events
**Plans**: 3 plans

Plans:
- [x] 05-01: /agents: live breadcrumb feed + 50-run history with per-run cost
- [x] 05-02: /digest: windowed engine timeline (overnight/24h/7d)
- [x] 05-03: Spend section on /digest: per-project totals + per-run costs, honest about unpriced

### Phase 6: Guard & Host Health
**Goal**: Eli sees what the guard decided on his behalf and whether the machine itself is healthy — the ops picture that closes out mission control
**Depends on**: Phase 1
**Requirements**: GUARD-01, HLTH-01
**Success Criteria** (what must be TRUE):
  1. Eli sees guard decisions (allow / block / ask) with reasons, parsed from ops/logs/guard.log
  2. Eli filters guard decisions (e.g. by decision type) and the list narrows accordingly
  3. Eli sees a health panel: DB reachable, Discord bridge alive, work-loop heartbeat, disk
  4. The health panel visibly carries the D-05 caveat that the dashboard shares the mini's fate
**Plans**: TBD (planned during plan-phase)

## Progress

| Phase | Plans Complete | Stage | Status |
|-------|----------------|-------|--------|
| 1. Projects at a Glance | 4/4 | verified | In progress |
| 2. Project Drill-down & Wave Lanes | 3/3 | verified | In progress |
| 3. Blocked-on-You Queue | 2/2 | verified | In progress |
| 4. Telemetry & Live Agents | 3/3 | verified | In progress |
| 5. History, Spend & Overnight Digest | 3/3 | verified | In progress |
| 6. Guard & Host Health | 0/0 | todo | Not started |

