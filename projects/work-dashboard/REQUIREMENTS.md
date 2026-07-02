# Requirements: Work-engine Dashboard

**Defined:** 2026-07-02
**Core Value:** Eli can see, at a glance and without asking, what the machine is doing, what it did, and what's blocked on him.

> Project: [[PROJECT]] · Roadmap: [[ROADMAP]]. Traceability is rendered from `momo_work`.

## v1 Requirements

### Views (projects & work)

- [ ] **VIEW-01**: Eli sees all projects with status and progress (done/doing/todo rollups).
- [ ] **VIEW-02**: Eli drills into a project: milestone → phases → plans → tasks with stage, status, and requirements coverage.
- [ ] **VIEW-03**: Eli sees a phase's plans as wave lanes (execution-order columns, colored by status) with a blocked-by list per plan.

### Agents

- [ ] **AGENT-01**: Eli sees agents running right now: host, project, plan/task, started-at.
- [ ] **AGENT-02**: Eli sees a live breadcrumb feed of a running agent's recent actions.
- [ ] **AGENT-03**: Eli sees run history: past agent runs with outcome, duration, and cost.

### Queue (blocked on Eli)

- [ ] **QUEUE-01**: Eli sees his open gates + human tasks, ranked by how much each blocks.
- [ ] **QUEUE-02**: Each queue item shows exactly which downstream plans/tasks it is holding up.

### Digest

- [ ] **DIG-01**: Eli sees a chronological timeline of engine events (plans completed, gates cleared, commits, DMs sent, spend), filterable by time window ("overnight").

### Usage (telemetry — new engine capability)

- [ ] **USE-01**: Agent runs record telemetry to the work DB: run start/end, host, plan/task, model tier, tokens, cost, breadcrumb events.
- [ ] **USE-02**: Eli sees spend per project over time.
- [ ] **USE-03**: Eli sees spend per agent run.

### Guard

- [ ] **GUARD-01**: Eli sees guard decisions (allow / block / ask, with reason), filterable — surfaced from the guard log.

### Health

- [ ] **HLTH-01**: Eli sees a best-effort health panel: DB reachable, Discord bridge alive, work-loop heartbeat, disk — with the explicit caveat that the dashboard shares the mini's fate (D-05).

## v2 Requirements

- **ACT-01**: Approve gates from the dashboard (interaction model validated first).
- **ACT-02**: Add/edit tasks from the dashboard.
- **VIEW-04**: Full dependency graph (arrows) view with critical-path tracing.
- **RMT-01**: Remote access (deployment + auth, decided together).

## Out of Scope

| Feature | Reason |
|---------|--------|
| Write actions in v1 | Interaction model unvalidated; Discord is the action channel (D-02) |
| Internet exposure / auth | Local-only (D-01); exposure and auth are one decision, later |
| Gantt chart | Engine is estimate-free by design; fake durations lie (D-03) |
| Standalone uptime alerting | Dashboard dies with its host (D-05); alerting lives elsewhere |

## Traceability

(Rendered after roadmap materialisation.)

---
*Requirements defined: 2026-07-02 (from Discord scoping with Eli)*
