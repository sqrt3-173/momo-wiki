# GSD → Work Engine — native replication design

The concrete translation of [[gsd-methodology|GSD]] onto the [[work-engine|Work engine]] (Postgres
`momo_work` + wiki + orchestration). Goal: **full faithful replication of GSD's methodology**, DB-backed,
always-on, multi-project — the same peak-reliability experience as GSD's `/commands`, staged core-first
toward complete parity. Started 2026-07-02.

## Principle
GSD is files + prompts + a deterministic engine, run in one Claude Code session. We keep the **methodology
100% faithful** (loop, docs, gates, verification, fresh-context orchestration) but re-seat it: **state → DB**
(queryable, atomic, multi-host), **knowledge docs → wiki**, **execution → the orchestrated work loop**.

## 1. Structure mapping (GSD granularity → tables)
GSD: `Project → Milestone (vX.Y) → Phase → Plan → Task`, with `Module` as a cross-cutting functional axis.

| GSD level | Work-engine table | Notes |
|---|---|---|
| Project | `projects` ✅ | + `config` (gates/workflow toggles) |
| Milestone | `milestones` ✅(dim) | vX.Y; usually 1; `target_date`, status |
| Phase | **`phases`** (new) | integer; decimal = urgent insertion; carries methodology stage + gate state |
| Plan | **`plans`** (new) | the *execution unit* — vertical slice, 2–3 tasks, one fresh agent, `wave`, `depends_on`, `files_modified`, `must_haves`, `requirements[]` |
| Task | `tasks` ✅ | steps inside a plan, acceptance-criteria hard gate; = GSD "task" |
| Module | `modules` ✅ | cross-cutting functional tag (`module_id` on plan/task) |

So the spine becomes `project → milestone → phase → plan → task`. A Work-engine unit an agent **claims + runs = a Plan** (its tasks are the checklist inside). Rework: rename current `tasks.parent_task_id`/subtask usage to sit under plans; `claim_next_task` → `claim_next_plan` (agent claims a ready plan in a ready wave).

## 2. Document mapping (GSD docs → three layers)
- **Knowledge → wiki** (`nunu-wiki`/`momo-wiki` or a project space): `PROJECT.md`, `REQUIREMENTS.md` (REQ-IDs), `research/*`, per-phase `CONTEXT.md`, `RESEARCH.md`, `UI-SPEC.md`, `SUMMARY.md`, `VERIFICATION.md`, `UAT.md`. Same templates, versioned by git.
- **Work → DB:** `ROADMAP.md` = a *rendered view* of the milestones/phases/plans rows (not a source file); `PLAN.md` frontmatter (`wave, depends_on, files_modified, autonomous, requirements, must_haves`) = `plans` columns; `STATE.md` = **replaced** by live `project_status`/`phase_status` views.
- **config.json → `project_config`** (gates + workflow toggles per project).

## 3. Lifecycle mapping (the phase loop → work-loop methodology)
Each GSD command = a **methodology step** the engine runs (an agent invocation with GSD's actual prompt), producing its doc + advancing DB state. Faithful to GSD's exact steps (see [[gsd-methodology]]).

- **new-project** → deep-questioning (human) → optional research (4 fresh researchers → synthesizer) → REQUIREMENTS → **roadmapper** materialises milestones/phases/plans **as rows** + writes ROADMAP view. Gates as human tasks.
- **discuss-phase** → scout + Q&A → `CONTEXT.md` (decisions `D-01…`, `canonical_refs`) → phase.stage='discussed'.
- **ui-phase** (frontend) → `UI-SPEC.md`; deterministic gate blocks planning without it.
- **plan-phase** → researcher → planner → plan-checker (revision loop max 3); materialises `plans` rows + PLAN docs; Requirements + Decision coverage gates (block).
- **execute-phase** → group plans into **waves** (deps met + no `files_modified` overlap) → per plan a fresh worktree-isolated executor → `SUMMARY.md` + plan.status=done; between-wave merge + build/test gate; then **verifier** (goal-backward `must_haves` vs real code).
- **verify-work** → human UAT tasks → issues → debug agents → planner `--gaps` → execute `--gaps-only`.
- **ship** → preflight gates → PR. (`complete-milestone` archives.)

## 4. Gates → human-in-the-loop (already built ✅)
GSD gates (`confirm_project`, `confirm_roadmap`, `confirm_breakdown`, `confirm_plan`, `execute_next_plan`, `issues_review`, `confirm_transition`) → each a **human task** (assignee=eli) the next phase/plan `depends_on`. `--auto`/`--chain` = a per-project config flag that auto-completes gate tasks. Safety gates (destructive/external) stay hard.

## 5. Orchestration → the work loop + agents
- **Fresh-context subagents** = GSD's anti-context-rot rule → I already spawn agents with paths-not-contents; the orchestrator (work loop) stays lean.
- **Waves** = the `plans.wave` grouping (pre-computed from deps + file-overlap) → parallel agent fan-out per wave, sequential between waves.
- **Deterministic engine** (`gsd-tools.cjs`) → **SQL functions on `momo_work`** (`claim_next_plan`, phase/roadmap views, wave grouping) — the DB *is* the deterministic layer.
- **35 subagent roles** → faithful agent prompts (researcher, planner, plan-checker, executor, verifier, roadmapper, ui-*, security-auditor, debugger, …), reconstructed from GSD's `agents/*.md`. Built core-first, rest via parallel reconstruction.

## 6. Build stages (core-first → full parity)
1. **Schema** — add `phases`, `plans`, `project_config`; rework claim to `claim_next_plan`; wave-aware readiness; migrate the existing task/subtask model under plans. *(next)*
2. **Planning flow** — `new-project` + `plan-phase`: questioning → research → REQUIREMENTS → roadmapper → materialise phases/plans; the core agent roles (researcher/synthesizer/roadmapper/planner/plan-checker) with GSD's real prompts.
3. **Execution loop** — `execute-phase`: wave dispatch of fresh executors (worktree-isolated) → SUMMARY → verifier (goal-backward) → UAT gates.
4. **Ship + milestone** — ship preflight/PR, complete-milestone.
5. **Full parity (orchestrated reconstruction)** — fan out agents to faithfully rebuild the remaining commands, templates, agent prompts, and references from GSD source (security-audit/ASVS, TDD, debug/forensics, brownfield map/ingest, spikes, mempalace, stats). Each verified against the source.

## Fidelity method
For each command/agent/template we replicate: read the GSD source file(s) → reproduce its prompt/steps/structure one-for-one → adapt only the *substrate* (files→DB, session→work-loop), never the *methodology*. Flag any genuine gap (e.g. `research.md` skeleton, some agent bodies not yet opened) and close it by reading that source when we build that piece.
