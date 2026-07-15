# MOMO Orchestration Architecture — master → sub-MOMO + cockpit

> **Status (2026-07-09, end of build session):** Core decisions LOCKED (see §5 RESOLVED).
> **Phase 1 (Observe) is BUILT, verified, and complete** — live process list, `.planning` file
> viewer, project→phase→instance drill-down, and live SSE transcript tail, all in the `:3100`
> dashboard, 160/161 tests green (the 1 failure is an unrelated pre-existing ingestion-db test).
> The build ran fully autonomously through the **sub-MOMO spawner** (`ops/momo-spawn.sh` +
> guardian hook): drop a request in `ops/spawn-requests/*.json` → guardian launches a guarded
> `claude -p` worker (from `/Users/momo/momo` so the guard loads) that follows the GSD workflow
> markdown (NOT the Skill tool — denied headless) → commits + verifies. Proven for both
> plan-phase and execute-phase.
>
> **Update (2026-07-09 evening, fresh session post-reset):** Phase 2 (Supervise) PLANNED (6
> plans, checker-passed — its discuss requirement was satisfied by the day's design session,
> reconciled in cockpit PROJECT.md commit f71ff39) and EXECUTING via spawner run 80: Wave 0
> (verification) done + committed; run stopped cleanly at the Wave-1 live-DB migration
> (`worksystem/013-control-commands.sql`) — **awaiting Eli's approval** (notification #19,
> delivered). On approval: apply migration, re-fire execute-phase-2.
>
> **PENDING Eli:** (1) UAT Phase 1, or move on (asked); (2) the 013 migration approval
> (asked, blocking waves 1–5); (3) team-granularity (single vs multi-agent teams) — needed
> before Phase 3 UI shape; (4) the 2026-07-09 injection guard-fix — now AUTHORED at
> `ops/patches/2026-07-09-subagent-control-files.patch`, batch with 02-06's guard patch for
> one sudo sitting; (5) FORGE gates re-delivered (FORGE-05 device scan = phase exit,
> FORGE-03 minimise.mov eyeball).
> **Interim spawner caveats:** headless `-p` can't run Skills or be typed into; launcher scripts
> not yet in guard PROTECTED_FILES (02-06 closes this); planning/execute runs are usage-heavy.

## 1. The vision (Eli, 2026-07-09)
A **master MOMO** that delegates to **sub-MOMOs**. Each sub-MOMO handles one GSD phase, then
clears when the phase is done. GSD is linear, so go phase-by-phase. Hit a blocker → don't
limp on in the same sub-MOMO; **kill it and spawn a fresh one** with a carry-on prompt.

The centrepiece is a **cockpit**: a visual terminal showing master + sub-MOMOs working, **true
to what is actually happening on the device** — not a parallel status store that can drift. It
surfaces the GSD `.planning/` files per project, lets Eli manage cron jobs and per-agent
`agent.md` prompt files, and — eventually — could replicate Claude-chat's "projects with
instances underneath" so MOMO spawns its own messaging channels (a tidier alternative to
Discord). "The main important bit" (Eli's words): the sub-agent architecture must be **visible
and faithful to reality.**

## 2. Current state — what already exists (verified 2026-07-09)
We are not starting from zero. The bones of this are already running.

**The engine (a working, unmanaged version of "master spawns workers"):**
`cron (60s) → ops/momo-guardian.sh → ops/momo-tick.sh → claude -p (headless) → receipts in
Postgres momo_work`. Every 30 min (throttle via `ops/locks/tick.stamp`; force via
`tick.force`) a tick picks ONE unit of work on ONE project, claims it with a lock file in
`ops/locks/`, runs a headless `claude -p` session (**Sonnet**, `--max-turns 100
--max-budget-usd 5`, 2700s wall-clock kill), and records a run row + step breadcrumbs in the
DB. Single-flight enforced (`LIVERUNS>0 → exit`). Circuit breaker halts the GSD scan after 2
consecutive errored runs until `ops/locks/gsd-breaker.reset`.

**The claim convention (already the "one worker per project" primitive):** a file in
`ops/locks/` whose name contains the project name = that project is owned. Interactive sessions
win; ticks skip claimed projects; the wrapper cleans its own dead run's claim on exit by
matching `RUN_ID`.

**The dashboard (already a read-only observability viewer) — LIVE on `:3100`:** Next.js 16 /
React 19, served self-healing via `ops/dashboard-guardian.sh` inside tmux `forge-dash`. Reads
`momo_work` through a **read-only DB role** (`dashboard_ro`, privilege-enforced, tested) plus
live system probes (`ops/logs/guard.log`, `pgrep` the bridge, disk, DB heartbeat). Routes:
`/` (project grid + status strip), `/projects/[slug]` (milestone→phase→plan→task drill-in,
**new/untracked**), `/queue` (human queue), `/agents` (live agents + run history +
breadcrumbs), `/digest` (event timeline + spend), `/system` (health + guard-log viewer).
Honest degradation model (a failed probe is "unknown", never "ok"). **It is purely a viewer —
no control endpoints. That gap is exactly what "cockpit" adds.**

**The database `momo_work` (already the state backbone):** DDL in `worksystem/{schema,functions,
002-012}*.sql`. Key tables: `projects, milestones, phases, plans, tasks, agent_runs,
agent_events, notifications, engine_state, inbox_items, atoms, seeds`. Key views:
`project_status, phase_status, live_agents, recent_runs, run_history`. Functions:
`loop_precheck(), start_run(), finish_run(), reap_stale_runs(), claim_next_plan(),
human_queue(), capture_item(), materialize_atoms()`. Runs carry
`host, kind, model_tier, started_at, heartbeat_at, ended_at, outcome, tokens, cost_usd, note,
project_id`; `live_agents` computes a `stale` flag from `heartbeat_at`.

**GSD project layout (the per-project `.md` files the cockpit surfaces):**
`projects/<name>/.planning/` holds `STATE.md` (YAML frontmatter the tick parses for
actionability + prose sections), `ROADMAP.md`, `PROJECT.md`, `REQUIREMENTS.md`, `research/`,
`codebase/`, `graphs/`, and `phases/<FORGE-NN-slug>/` dirs containing `NN-NN-PLAN.md` /
`NN-NN-SUMMARY.md` / `NN-RESEARCH.md` / `NN-VALIDATION.md` / `VERIFICATION.md` / `evidence/`.
Human/device checkpoints surface as `notifications` rows (kind `gsd-device-ask` etc.), not as
hard blocks.

**The capability ladder (enforced by `ops/momo-guard.py`, keyed on `agent_id`):**

| Actor | Bash/psql | Code writes | `.planning/*.md` | Web | Discord | DB writes |
|---|---|---|---|---|---|---|
| In-process subagent (`agent_id`) | ❌ | ❌ | ✅ carve-out | ✅ | ❌ | ❌ |
| Headless `claude -p` (own guard, no `agent_id`) | ✅ allowlist | ✅ work dir | ✅ | ❌ (delegates) | ❌ (→`notifications`) | ✅ additive |
| Interactive session | ✅ | ✅ | ✅ | delegates | ✅ sole bridge | ✅ |

This ladder is the crux of the architecture decision below.

## 3. Recommended target architecture
**A sub-MOMO = an independent `claude -p` session, NOT an in-process Agent-tool subagent.**
This is the single decision that shapes everything, and the table in §2 is why:

- **Visible & true-to-reality** — an independent session is a real OS process with its own
  transcript file. The cockpit shows *the actual thing*, nothing to fake or reconcile.
- **Capable** — it runs under the main-loop guard tier, so it can actually execute a GSD phase
  (Bash, code writes, DB). An in-process subagent is locked to web+read+`.planning`-md and
  literally cannot execute a phase.
- **Controllable** — master can spawn, monitor (heartbeat/transcript), and kill it for real.
- **Blocker → kill → respawn-fresh** maps to a clean process lifecycle: no shared mutable
  in-memory state to corrupt; the new session rebuilds from disk (`.planning/` + DB).
- **Security: designs out the 2026-07-09 injection.** Each sub-MOMO has its own guard and its
  own report channel, so none can get cornered into planting instructions in a shared file —
  the exact trap that produced [[../ops/incidents/2026-07-09-state-injection/README|that
  incident]]. In-process subagents are the actor that caused it.

**Master's role:** owns the roadmap-level view, decides which phase runs next (linear by
default), spawns the sub-MOMO with a scoped prompt + `agent.md`, watches its heartbeat, and on
completion/blocker tears it down and either advances or respawns. This is the current guardian/
tick loop **promoted from an unmanaged cron drip into a supervised, visible manager.**

**The claim/lock convention and `momo_work` run rows already give us** the "one owner per
project," "live vs stale," and "run history" primitives — the manager layer formalises them
rather than inventing new state.

## 4. The cockpit
**Principle (Eli's "main important bit"): the cockpit reads the exact artifacts the system runs
on — live processes, transcripts, `ops/locks/` claims, `.planning/`, `momo_work` — never a
separate status store that can drift.** If it's shown, it's true on disk.

Build **on the existing dashboard**, adding two things it lacks:
1. **Sub-MOMO visibility** — live process list (master + each sub-MOMO), each with its current
   phase, heartbeat, budget/turns burned, and a **live transcript tail** (the real session
   `.jsonl`). Plus the `.planning/` file tree per project, rendered inline.
2. **Control plane** — the viewer is read-only by DB privilege today; controls need a
   *separate, audited* write path (never loosen `dashboard_ro`): pause/resume the loop, force a
   tick, spawn/kill a sub-MOMO, edit an `agent.md`, manage cron entries. Every control action
   goes through the guard + an audit row.

Structure mirrors Eli's "projects with instances underneath": project → its phases → the
sub-MOMO instance(s) that ran/are running each → click in to watch the real transcript.

## 5. Open decisions
**RESOLVED (Eli, 2026-07-09):**
1. ✅ **Independent-sessions model** (§3) — confirmed. A sub-MOMO is a real `claude` process (not an in-process Agent-tool subagent).
2. ✅ **Web app** cockpit — extends the existing `:3100` dashboard.
3. ✅ **Sub-MOMOs are INTERACTIVE talk-to-able tabs** (refines #1) — interactive `claude` sessions in PTY tabs (node-pty + xterm.js, ADE-style), NOT headless-only; master-driven (programmatic input) AND Eli can jump into any tab and take over; guard still loads (non-bare) so interactive ≠ unguarded. Headless `claude -p` remains for fully-autonomous background jobs (the bootstrap spawner). **Interactive is also what lets a tab run GSD slash-commands** (`/gsd-new-project`, `/gsd-plan-phase`, …) — headless `-p` denies Skills (proven tonight, spawn run 57).
   - **SPIKE RESULT (2026-07-09, claude-code-guide agent):** concurrent program-driven + human input to the SAME live session is NOT supported (Claude Code takes input from one source at a time; `-p` is one-shot; the Agent SDK is programmatic-only; `--resume` is sequential not concurrent). So the buildable model is **"watch always + take over on demand":** the master drives each stage headless / workflow-following (read-only live transcript to the tab), and when Eli wants in he hits *take over* → the cockpit opens that exact session interactively via `claude --resume <session-id>` (full context) → he types / runs slash-commands → hands back. Sequential hand-off, not a shared keyboard. This still delivers Eli's ask (see + intervene on any sub-MOMO) and doesn't touch Phase 1 (pure-watch); it shapes Supervise/Control.
4. ✅ **Org hierarchy** — **Business/client** (NV Health, Forge) → **Team = a *function*** (website builder, social-media manager, Google Ads, accounting — NOT a whole business) → **Agent(s)** (a team is usually one agent; the structure supports several) → **Tabs = GSD stages** (each stage/command runs in its OWN fresh interactive tab, opened *alongside* the previous ones, so the whole build progression is visible and any stage is openable to watch/talk to). **Master MOMO = PM over all of it** — kicks off stages, spawns the next stage's tab on completion, reviews the work.
5. ✅ **Fresh-tab-per-stage = fresh-context-per-stage + a visible build history** — each GSD command gets a clean session in its own tab; old tabs persist as the scrollable history of how the project got built.

**Open (Eli to confirm):** team granularity — single-agent teams vs multi-agent teams (recommended: support both).

**Secondary (resolve during design):**
3. Control-plane write path — a small authenticated API the cockpit calls, distinct from the
   read-only viewer role. How does master actually receive a "spawn/kill" command (DB command
   row the guardian polls? direct API?).
4. `agent.md` per sub-MOMO — where they live, how master injects them into the spawn prompt.
5. Discord replacement (Eli's "spawn own channels") — **sequenced LAST** (see §6). Keep Discord
   as comms until the cockpit is proven.

## 6. Phased build plan (proposal)
1. **Observe** — extend the dashboard to show live sub-MOMO processes + transcript tails +
   `.planning/` trees. Pure read, no new risk. Delivers the "see what's actually happening"
   core immediately.
2. **Supervise** — promote the tick loop into a real master/manager: explicit spawn of a
   scoped sub-MOMO per phase, heartbeat monitoring, blocker → kill → respawn-fresh.
3. **Control** — add the audited control plane (pause/force/spawn/kill/edit-agent.md/cron)
   behind the guard.
4. **(Later) In-app comms** — MOMO-spawned channels as a Discord alternative. Only after 1–3
   are proven.

## 7. How this supersedes the injection incident
The [[../ops/incidents/2026-07-09-state-injection/README|2026-07-09 injection]] happened because
an in-process subagent, restricted to `.planning`-md writes and with no report channel, planted
an instruction into a file a trusted tick reads. Moving execution to **independent sub-MOMO
sessions** removes both preconditions: each has full capability and its own report channel, so
it never routes a plea through shared data. The guard-carve-out fix (exclude control files;
read `STATE.md` as inert data) remains worth doing as defense-in-depth for the research/plan
subagents that still exist — see the incident's follow-ups.
