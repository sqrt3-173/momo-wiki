# Work-engine Dashboard (Mission Control)

> GSD-style living project context. Scoped with Eli over Discord 2026-07-02 (deep-questioning
> step of new-project). Requirements: [[REQUIREMENTS]] · Roadmap: [[ROADMAP]].

## What This Is

A local, read-only web dashboard for the work engine — mission control for everything the
machine does. One screen answers "what's happening / what happened while I slept / what's
waiting on me": projects with their phases, plans and tasks; agents working live; Eli's queue;
an overnight digest; usage/cost; guard decisions; host health.

## Core Value

Eli can see, at a glance and without asking, what the machine is doing, what it did, and what's
blocked on him.

## Requirements

### Active

See [[REQUIREMENTS]] — 14 v1 requirements across VIEW / AGENT / QUEUE / DIG / USE / GUARD / HLTH.

### Out of Scope

- **Any write action** (approving gates, adding tasks) — v1 is read-only; Eli validates how he
  wants to interact with it before we add controls. Discord stays the action channel.
- **Deployment / internet exposure** — local network only for now; no auth layer needed yet.
  Revisit both together (exposure requires auth).
- **Gantt chart** — the engine has no time estimates by design; wave lanes + dependency list
  instead (D-03).
- **NUNU anything** — retired.

## Constraints

- **Tech stack**: Next.js/React + shadcn/ui — house stack; candidate build aids (shadcn MCP +
  skill, third-party dashboard-layout skill) being vetted, installs gated on Eli.
- **Runs on the mini**, reads `momo_work` Postgres directly (localhost) + `ops/logs/guard.log`.
- **Read-only DB access** — the app gets a read-only view of the engine; it must never mutate work state.
- **Tests written** (house rule).

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| D-01 Local-only, no deployment | Zero exposure; work DB stays private; Eli uses MacBook on home network | — Pending |
| D-02 Read-only v1 | Validate interaction model before building controls; Discord already handles actions | — Pending |
| D-03 No Gantt — wave lanes + blocked-on-you list | Engine is estimate-free by design; fake durations lie | — Pending |
| D-04 Telemetry layer is part of this build | Usage/digest/agent-feed views all need events the engine doesn't record yet | — Pending |
| D-05 Health panel is best-effort, not monitoring | Dashboard dies with the mini it runs on; real alerting must live elsewhere | — Pending |

---
*Last updated: 2026-07-02 at project creation.*
