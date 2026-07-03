# Phase 4 Research: Heartbeat Loop

**Phase:** 4 — Heartbeat Loop (ingestion-loop)
**Researched:** 2026-07-03
**Domain:** Scheduled autonomous agent loop on macOS — cron/launchd trigger, shell pre-check,
headless `claude -p` sessions under the PreToolUse guard, Postgres claim/telemetry, Discord-decoupled reporting
**Overall Confidence:** MEDIUM-HIGH (CLI mechanics verified against current official docs; local substrate
read directly; two load-bearing behaviors need a live probe the researcher's tool policy blocked — flagged)

> Researcher note on probes: this agent ran under the guard's subagent lockdown — Bash was blocked
> (`SUBAGENT-BLOCKED`), so no environment probes (`claude --version`, `pgrep`, `crontab -l`, psql reads) ran.
> Everything probe-dependent is tagged [ASSUMED] with a required verification task. All local file reads are
> [VERIFIED: file] — read directly this session.

## User Constraints

**Locked decisions (never recommend against):**
- Substrate: `momo_work` Postgres 16 (local), wiki, Discord via the plugin's reply tool, PreToolUse guard
  (`ops/momo-guard.py`), telemetry trio (`start_run`/`log_event`/`finish_run`), dashboard. SQL migrations
  `worksystem/NNN-*.sql`; runbooks + agent prompts as markdown. **No new services/SaaS.**
- Idle must cost ~nothing: shell pre-check BEFORE any Claude session wakes (D-04 cheap-check-first).
- The guard governs tick sessions exactly as the main loop (same settings, same categories).
- Standing guard exemption (Eli 2026-07-03, decision 4-2): INSERT/UPDATE on momo_work auto-approved
  (inline `-c`, target db provable, no destructive/DDL verbs anywhere in the command — fail-closed).
  DELETE/DROP/TRUNCATE/ALTER/CREATE and `psql -f` still ask → tick sessions can do routine engine writes
  unattended but CANNOT run migrations or destructive SQL.
- launchd on this Mac pends automatic spawns; the guardian pattern is the proven survivable trigger.
  Installing a new launchd agent is ASK-ELI. (Research finding below sharpens this: the proven trigger is
  **cron**, not launchd StartInterval — see Pitfall 1.)
- Discord bridge is bound to the interactive `momo` tmux session. Eli's "pause"/"resume" arrives in the
  INTERACTIVE session — the flag write happens there; the pre-check just reads it.
- Eli's governing goal (verbatim): "a smart orchestration of agents to work through projects, keep me in
  the loop, but most importantly be quite autonomous, and always working on something, and having critical
  thinking for giving suggestions that expand beyond current idea set for me to approve and breakdown even
  more." Autonomy + always-working ranks first; suggestions route to his gate.
- Economics: runs unpriced (no Fable 5 rate recorded, DL-17); no run-rate ceiling set. Phase must include
  per-tick cost visibility (tokens recorded even if unpriced) and a conservative default posture
  (max 1 unit/tick, ~30-min interval, hard skip if a run is already live).
- LOOP-05 (parallel fan-out) explicitly deferred — out of scope.

**Claude's discretion:** exact tick interval constant, pre-check query shape, lock mechanism choice,
notification-drain trigger, prompt-file structure, watchdog timeout values.

**Deferred (ignore):** parallel fan-out, auto-approval of hard-gated categories, STT.

## Phase Requirements

| REQ | Description | Enabled by findings |
|---|---|---|
| LOOP-01 | Scheduled tick ~30 min, shell pre-check, near-free idle | Guardian-hosted stamp-throttled tick (Arch §1); single `loop_precheck()` SQL round-trip (Arch §3); pause flag read in the same query |
| LOOP-02 | Wake fresh Claude session, claim ONE unit, full discipline, clean close | `claude -p` non-bare inherits guard/hooks/CLAUDE.md (Finding H1); wrapper-owned run row + watchdog (Arch §4); gate-aware claim fix (Pitfall 4); headless runbook divergences (Arch §6) |
| LOOP-03 | pause/resume flag from Discord | `engine_state` table (Arch §5) — interactive session UPDATEs it (covered by 4-2 exemption); pre-check reads it; feeds VIS-02 via dashboard_ro |
| LOOP-04 | Telemetry + digest for every tick session | Wrapper calls `start_run` BEFORE launch and `finish_run` after with tokens/cost from `--output-format json` (Finding H4) — telemetry by construction, even on crash; outcomes land as `log_event` breadcrumbs the digest view already surfaces |

## Architectural Responsibility Map

| Capability | Owning tier | Rationale |
|---|---|---|
| Trigger (every 60s, survive reboot/kill) | **cron → `ops/momo-guardian.sh`** (existing) | The ONLY empirically surviving scheduler on this Mac (ops/README.md, 2026-07-02); no new install surface |
| 30-min throttle + idle pre-check + single-flight lock + watchdog | **`ops/momo-tick.sh`** (new shell script, backgrounded by guardian) | Must run BEFORE any Claude session (D-04); shell is free; deterministic code, not judgment |
| Work detection, pause flag, unit priority, stale-run reaping | **Postgres `momo_work`** (`loop_precheck()`, `engine_state`, `reap_stale_runs()`, gate-aware claim) | State + atomicity live in the DB (engine invariant); one round trip keeps idle near-free |
| Unit execution (triage batch / one plan / seeds review) | **Headless `claude -p` session**, cwd `/Users/momo/momo`, governed by its own guard instance | Reasoning work; must inherit full pipeline discipline (guard, runbooks, telemetry) |
| Discord I/O (DMs out, pause/resume in) | **Interactive `momo` tmux session ONLY** | The bridge is bound to it; a second bridge breaks replies for both (discord-bridge.md incident, 2026-06-22/07-02) |
| Durable outbound messages from headless runs | **`notifications` queue table** in momo_work | Headless sessions have no reply tool; a DB row survives; the interactive session drains it |
| Receipts + visibility | **Telemetry trio + wiki + dashboard** (existing) | LOOP-04 / VIS-02 substrate already built |

## Summary

The phase is glue, not invention: every hard component already exists and is proven on this machine. The
trigger is the cron-driven guardian (the one scheduler that survives on this macOS build — launchd
StartInterval was ALSO pended, correcting the phase brief's assumption). The pre-check is one STABLE SQL
function returning pause-flag + work counts + live-run count in a single psql round trip, throttled by a
stamp file so 29 of every 30 guardian passes never touch the DB. When work exists, a wrapper script — not
the model — creates the telemetry run row, launches a **non-bare** `claude -p` from `/Users/momo/momo`
(non-bare is load-bearing: `--bare` would skip settings auto-discovery and therefore drop the PreToolUse
guard), bounds it with `--max-turns` + a shell watchdog, and closes the run row with tokens and cost parsed
from `--output-format json`. The session claims exactly one unit and executes the existing runbook for it;
anywhere a runbook says "DM Eli," the headless variant inserts a `notifications` row instead, because the
Discord bridge belongs exclusively to the interactive session.

The crux distinction the brief asked for, resolved: the runbooks' "NEVER `claude -p` from bash (IL-02)"
governs a **Claude session spawning nested sessions through its own Bash tool** — the guard blocks `claude`
as an unknown leading tool, correctly, and that stays. The scheduler is **not a Claude session**: cron →
guardian → tick script runs outside any hook, so it may exec `claude -p` directly; the spawned session then
reads `.claude/settings.local.json` from its cwd and gets its own guard instance governing every tool call
it makes (its own subagents even inherit the subagent lockdown via `agent_id`). Two open hazards need a
live probe before wiring: whether a non-bare `-p` session spawns a duplicate Discord bridge process
(mitigation identified: `--strict-mcp-config` with no `--mcp-config` loads zero MCP servers), and whether
cron-context OAuth resolves (strong evidence yes — the guardian already launches the interactive claude
from cron and it authenticates).

**Primary recommendation:** Host the tick inside the existing guardian via a stamp-file throttle that
backgrounds a new `ops/momo-tick.sh`; one `loop_precheck()` SQL call decides idle/paused/work; wake
non-bare `claude -p --strict-mcp-config --max-turns 40 --output-format json` with wrapper-owned telemetry
and a shell watchdog; route all headless outbound messages through a `notifications` table the interactive
session drains — and make plan 1 of this phase an empirical probe run that proves guard inheritance,
no-duplicate-bridge, and cron auth before anything is scheduled.

## Standard Stack

No packages are installed in this phase — everything rides tools already on the machine.

| Layer | Tool | Version | Role |
|---|---|---|---|
| Scheduler | cron (user crontab, existing entry) → `momo-guardian.sh` | macOS built-in | 60s single-pass reconciler, already live [VERIFIED: ops/README.md] |
| Tick logic | bash + `date` + stamp file + `mkdir` lock | macOS built-in | throttle, single-flight, watchdog |
| Pre-check | `psql` (Postgres 16, local) | existing | one `SELECT loop_precheck()` round trip |
| Session | `claude -p` (Claude Code CLI) | installed; version [ASSUMED ≥2.1.x — probe `claude -v`] | the governed work session |
| JSON parsing | `/usr/bin/python3` | macOS built-in | parse `--output-format json` result (avoids a jq dependency question; python3 is guard-allowlisted and provably present — the guard itself runs on it) |
| State | `momo_work` migration `worksystem/012-heartbeat.sql` | — | `engine_state`, `notifications`, `loop_precheck()`, `reap_stale_runs()`, gate-aware claim |

**Key CLI flags (all [VERIFIED: code.claude.com/docs/en/cli-reference, fetched 2026-07-03]):**

| Flag | Use here |
|---|---|
| `-p` / `--print` | headless run, exit on completion |
| `--output-format json` | result payload includes `total_cost_usd`, per-model cost breakdown, `session_id`, usage metadata → per-tick cost visibility [VERIFIED: headless docs] |
| `--max-turns N` | hard turn bound, print-mode only, "exits with an error when the limit is reached. No limit by default" — the runaway bound |
| `--max-budget-usd X` | dollar ceiling per invocation (print-mode only) — belt over braces; behavior under subscription OAuth unverified [ASSUMED — probe] |
| `--strict-mcp-config` (without `--mcp-config`) | "no MCP servers load" — the duplicate-bridge mitigation [CITED: cli-reference `--tools` row] |
| `--permission-mode dontAsk` | locked-down baseline: anything not explicitly allowed is denied rather than hung; the guard's ALLOW decisions still pass tools through [CITED: headless docs] |
| `--no-session-persistence` | optional: don't accumulate resumable transcripts from 48 ticks/day |
| `--fallback-model sonnet` | optional resilience if the primary model is overloaded overnight |

**DO NOT USE:** `--bare` — it "skips auto-discovery of hooks, skills, plugins, MCP servers, auto memory, and
CLAUDE.md" [VERIFIED: headless docs]. That would strip the PreToolUse guard AND soul.md/CLAUDE.md context.
Also skips OAuth/keychain (needs `ANTHROPIC_API_KEY` — a new billing surface Eli hasn't approved).
`--dangerously-skip-permissions` — never; the guard + `dontAsk` is the correct posture.

## Package Legitimacy Audit

Not applicable — zero packages installed. (This is itself a design assertion: any plan task that introduces
an install, e.g. GNU coreutils for `timeout` or `flock`, must instead use the built-in patterns in Code
Examples, or go to Eli as an ASK.)

## Architecture Patterns

### §1 Data flow (primary case traceable end to end)

```
cron (every 60s, existing entry)
  └─ momo-guardian.sh  (existing; add ~6 lines)
       ├─ [existing] backup + tmux session reconcile
       └─ [new] if now − mtime(ops/locks/tick.stamp) ≥ TICK_INTERVAL (30 min):
              touch stamp; nohup ops/momo-tick.sh & (guardian exits — stays single-pass)

ops/momo-tick.sh
  ├─ mkdir ops/locks/tick.lock  → fails = tick already running → exit 0        (single-flight #1)
  ├─ psql -t -A -c "SELECT loop_precheck();"                                    (ONE round trip)
  │     paused? → log, update engine_state.last_tick, exit           ← LOOP-03 respected here
  │     live run (fresh heartbeat)? → hard skip, exit                          (single-flight #2)
  │     stale live runs? → SELECT reap_stale_runs()  (mark 'killed', note)     (zombie cleanup)
  │     no work? → update last_tick, exit                            ← LOOP-01 idle ≈ 1 psql call
  ├─ pick ONE unit by priority: untriaged atoms → ready gated plan → seeds due
  ├─ RUN_ID=$(psql ... "SELECT start_run('mini','tick-<unit>',...)")   ← telemetry BEFORE launch
  ├─ cd /Users/momo/momo && claude -p "<tick prompt naming unit + RUN_ID>" \
  │       --output-format json --max-turns 40 --permission-mode dontAsk --strict-mcp-config \
  │       > out.json 2>err.log &   + shell watchdog (TERM at 45 min, KILL +30s)
  ├─ parse out.json (python3): result, total_cost_usd, usage tokens, is_error
  ├─ psql "SELECT finish_run(RUN_ID, <ok|error|killed>, <tokens>, <cost|NULL>, '<note>')"
  └─ update engine_state last_tick/last_result; rmdir lock; exit

claude -p session (fresh context, OWN guard instance from .claude/settings.local.json)
  ├─ reads the named runbook (triage.md / execute one plan / seeds-review.md)
  ├─ claims its one unit atomically in the DB; log_event breadcrumbs against RUN_ID
  ├─ writes outcomes: atom fates, wiki pages (+ commit; wiki push is guard-standing-approved),
  │   seeds updates — all inside guard-allowed categories (4-2 additive writes)
  └─ anything Eli-facing → INSERT INTO notifications (...)  — NEVER a Discord attempt

interactive momo session (the only bridge holder)
  ├─ "pause"/"resume" from Eli → UPDATE engine_state (4-2-covered)
  └─ drains notifications → reply tool → marks sent_at   (on next interaction + morning digest)
```

### §2 The two `claude -p` contexts — the crux, stated precisely

| Context | Governed by | Verdict |
|---|---|---|
| A Claude session (interactive or tick) running `claude -p ...` via its Bash tool | That session's PreToolUse guard: `claude` is not in `DEV_ALLOW` → `unknown` → ASK-ELI, **not marker-bypassable** [VERIFIED: ops/momo-guard.py `analyse_tokens` + line "unknown commands are NOT marker-bypassable"] | Blocked — and stays blocked. This is IL-02; also prevents tick fork-bombs by construction |
| cron → guardian → `momo-tick.sh` exec-ing `claude -p` | Nothing — hooks only exist inside Claude sessions; this is plain shell | Allowed and correct. The spawned session then loads `.claude/settings.local.json` from cwd → **its own** guard governs everything it does |

Non-bare `-p` "loads the same context an interactive session would, including anything configured in the
working directory or `~/.claude`" [VERIFIED: code.claude.com/docs/en/headless]. Hooks are read from
`.claude/settings.local.json` among other sources and run in print mode [CITED: code.claude.com/docs/en/hooks].
Subagent lockdown carries over: subagent calls carry `agent_id` → web/read only — so a tick session's triage
agent, extractor, and researchers are exactly as contained as the interactive session's [VERIFIED: momo-guard.py].
Consequence worth noting for the planner: main-loop WebFetch/WebSearch is ASK-blocked in the guard — tick
sessions must delegate all fetching to subagents, which the runbooks already mandate.

### §3 Pre-check: one function, one round trip

`loop_precheck()` (STABLE, read-only — passes the guard's psql read gate AND runs free from the ungoverned
tick script) returns a single row: `paused`, `pending_atoms`, `new_items`, `ready_plans` (gate-aware — see
Pitfall 4), `seeds_due` (function exists: `seeds_due_count()` [VERIFIED: worksystem/011-seeds.sql]),
`live_runs` (fresh heartbeat), `stale_runs`. Shell parses one delimited line. Idle cost: one local psql
connection every 30 min, zero Claude tokens — LOOP-01's "near-free" is literal.

### §4 Wrapper-owned telemetry (LOOP-04 by construction)

The wrapper calls `start_run` BEFORE launch and passes RUN_ID into the prompt; the session `log_event`s
against it (each call bumps heartbeat — dashboard liveness stays honest); the wrapper calls `finish_run`
with tokens/cost parsed from the JSON result and the process exit status. A session that dies instantly,
hangs, or gets watchdog-killed STILL has a complete telemetry record — "every tick session shows up in
telemetry" cannot be violated by session misbehavior. Cost: `tokens` from the JSON usage totals
digit-for-digit; `cost_usd` NULL while Fable 5 is unpriced (missing renders absent, never 0 — engine rule
[VERIFIED: wiki/ops/work-engine.md]).

### §5 Pause/resume + loop state: `engine_state` table, not `project_config`

`project_config` is per-project (`project_id` PK, JSONB) [VERIFIED: worksystem/003-gsd-schema.sql] — wrong
shape for an engine-wide flag. Use a new key-value table:

```sql
CREATE TABLE engine_state (
  key        TEXT PRIMARY KEY,          -- 'loop' | 'last_tick_at' | 'last_tick_result' | ...
  value      TEXT NOT NULL,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);  -- + GRANT SELECT TO dashboard_ro (feeds VIS-02: last tick / next expected / paused)
```

Interactive session on "pause": `UPDATE engine_state SET value='paused', updated_at=now() WHERE key='loop';`
— an UPDATE on momo_work, covered by the 4-2 standing exemption, so pause/resume works unattended-instantly.
A flag FILE was considered and rejected: the DB flag is dashboard-readable (VIS-02), atomic, and consistent
with "state lives in momo_work."

### §6 Headless runbook divergence (one rule, stated once)

The tick prompt states: *"You are a headless tick session. You have NO Discord tools. Wherever a runbook
step says to DM Eli / send via the reply tool / verify via fetch_messages: instead INSERT INTO notifications
(kind, body) and continue; the interactive session delivers and verifies."* The runbooks themselves stay
single-source; each gets a one-line "headless: see heartbeat runbook §notifications" note rather than a fork.
Held atoms (`HELD <date>:` prefix) work unchanged — the question text goes to the queue; Eli's answer arrives
in the interactive session, which owns the resolution path already (triage.md "On answer").

### §7 Unit priority (Claude's discretion, recommended)

1. **Untriaged atoms** (`fate='pending'`) — the overnight-dump promise (success criterion 5) and Eli-latency.
2. **Ready gated plan** (`claim_next_plan`, gate-aware) — "always working on something."
3. **Seeds due** (`seeds_due_count() > 0`) — weekly cadence, least time-critical.
One unit per tick, hard. A tick that finds nothing at its chosen priority does NOT fall through to a second
unit — it takes the highest non-empty category only.

### Anti-patterns to avoid
- **A second bridge anywhere.** Any process holding the Discord bot connection besides the interactive
  session splits inbound/outbound and breaks replies for both (proven twice: 2026-06-22, 2026-07-02).
- **The session reporting its own success as truth.** The digest reads receipts (DB rows, git log), never
  the session's prose claims (session-recovery.md's lesson).
- **Long-lived daemons.** Everything here is single-pass or bounded; nothing persists to be killed.
- **A new launchd job.** Empirically dead on this Mac in ALL automatic modes (see Pitfall 1) — and an
  ASK-ELI install for nothing.

## Don't Hand-Roll

| Problem | Don't build | Use instead | Why |
|---|---|---|---|
| Atomic unit claim | any SELECT-then-UPDATE claim logic | `claim_next_plan()` — `FOR UPDATE SKIP LOCKED`, proven 2-host-safe [VERIFIED: 003-gsd-schema.sql] (+ gate condition, Pitfall 4) | the DB already guarantees no double-claim |
| Runaway session bound | token counters, prompt-side "please stop" | `--max-turns` (+ `--max-budget-usd` if probe confirms) + shell watchdog | exits with an error at the limit — enforced by the harness, not by model compliance |
| Per-tick cost capture | scraping transcript text | `--output-format json` → `total_cost_usd` + usage | structured, documented fields [VERIFIED: headless docs] |
| Scheduler | new launchd plist, new daemon, a `while true; sleep 1800` loop | existing cron→guardian + stamp throttle | the only trigger proven to survive this macOS build; zero new install surface; zero new approval |
| Telemetry lifecycle | new logging tables | `start_run`/`log_event`/`finish_run` [VERIFIED: 008-telemetry.sql] | dashboard views already consume them |
| Seeds-due check | date math in shell | `seeds_due_count()` [VERIFIED: 011-seeds.sql] — built for exactly this pre-check | "Postgres computes dates, never the agent" (engine rule) |
| Session freshness | resuming/continuing tick conversations | fresh `-p` each tick (+ optionally `--no-session-persistence`) | GSD anti-context-rot rule; a tick has no legitimate memory |

**Key insight:** the phase adds ~2 files, ~1 migration, ~1 prompt — everything load-bearing (claiming,
gating, telemetry, scheduling, guard) already exists and is proven. Plans that invent more machinery than
this are over-scoped.

## Common Pitfalls

1. **launchd is dead on this Mac — in every automatic mode.** The phase brief said "StartInterval single-pass
   jobs ride the timer path and survive." The ops record says otherwise: "KeepAlive respawns, then RunAtLoad,
   then StartInterval ticks were ALL pended as 'inefficient'; only manual kickstart ran... single-pass
   reconciler triggered by **cron every minute** (verified ticking on schedule 2026-07-02)"
   [VERIFIED: ops/README.md; the guardian.sh header comment still says launchd/StartInterval — stale, the
   README's cron account is the corrected final architecture]. → Host the tick on the cron→guardian path.
   Avoid: any new plist. Warning sign: a plan task that says `launchctl bootstrap`.
2. **`--bare` silently disarms the guard.** It skips settings/hooks auto-discovery — a bare tick session
   would run WITHOUT the PreToolUse guard and without CLAUDE.md identity. Docs even note bare "will become
   the default for `-p` in a future release" — so **pin the behavior**: launch explicitly non-bare today,
   and the probe plan must assert the guard fired (read `ops/logs/guard.log` for the tick session's ALLOW
   lines) rather than assume it. Warning sign: a tick that never writes guard.log lines.
3. **Duplicate Discord bridge from `-p` sessions.** A plain interactive `claude` (no `--channels`) still
   spawns the plugin's bun bridge [VERIFIED: ops/README.md, guardian pgrep matches both forms]. Non-bare `-p`
   loads plugins/MCP the same way [CITED: headless docs] → a tick could spawn a colliding bridge, breaking
   Eli's replies AND deadlocking guardian restarts (its pgrep would see an "ad-hoc bridge" and stand by).
   Mitigation: `--strict-mcp-config` with no `--mcp-config` → "no MCP servers load" [CITED: cli-reference].
   MUST be proven live: launch one probe tick, `pgrep -U momo -f '<plugin bridge pattern>'`, confirm zero.
   `--disallowedTools "mcp__*"` alone is NOT sufficient — it removes tools from context but may not prevent
   the server process spawn.
4. **`claim_next_plan` ignores gates — the tick would amplify a latent gate bypass.** The function yields
   plans whose phase is `planned` OR `executing`; `plan-phase.md` keeps `confirm_plan` gates as human tasks
   and only flips stage to `executing` when the gate clears — but nothing in the SQL enforces it
   [VERIFIED: 003-gsd-schema.sql vs 005-gates-render.sql]. Interactively MOMO respects gates by discipline;
   an unattended tick has no discipline but code. → Migration 012 must make the claim gate-aware (e.g.
   `CREATE OR REPLACE claim_next_plan` adding `AND NOT EXISTS (open gate task for that phase/project)` via
   the `tasks.gate` machinery), and `loop_precheck()`'s `ready_plans` must count with the same condition.
   Warning sign: overnight digest shows a plan executed while its `confirm_plan` task is still open.
5. **The Stop hook fights headless sessions — bounded, but shape prompts anyway.** `stop-reply-guard.py`
   blocks any turn ending with >120 chars of prose and no Discord reply call; a tick session can never make
   that call. It honours `stop_hook_active`, so cost is exactly one forced continuation per offending stop,
   then it fails open [VERIFIED: ops/stop-reply-guard.py]. → The tick prompt instructs: end with a single
   line under 100 chars ("tick done: run <id>, <unit>, receipts in DB"). Do NOT edit the Stop hook or
   settings (root-owned, uchg, guard-hard-blocked).
6. **cron environment is not your shell.** No brew PATH, minimal env. The guardian already exports the full
   PATH; `momo-tick.sh` must do the same (claude lives at `/Users/momo/.local/bin` per momo-loop.sh) and
   `cd /Users/momo/momo` before launching (settings/guard/CLAUDE.md discovery is cwd-based). OAuth from
   cron context: strong empirical support (the guardian cron-launches the interactive claude and it
   authenticates) but assert it in the probe [ASSUMED→probe]. Warning sign: JSON result with an
   authentication error category.
7. **Zombie/hung sessions.** macOS has no `timeout(1)` or `flock(1)` [ASSUMED — no coreutils/util-linux by
   default; confirm with `command -v` in the probe]. Use the bash watchdog + `mkdir` lock patterns (Code
   Examples). Belt: `CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS` already caps stuck background-agent waits at
   10 min by default [VERIFIED: headless docs]. Stale telemetry runs get reaped by the NEXT tick
   (`reap_stale_runs()` marks heartbeat-silent runs `killed`) so the dashboard never shows phantom liveness.
8. **Notification loss.** Comparable cron+LLM builds lose reports when delivery is attempted at run time
   (process dies → message gone). Durable queue table + drain-on-next-interaction + morning digest solves
   it; the DM is a *view* of DB state, never the state itself. Also matches the engine's PERSIST FIRST rule.
9. **Runaway spend while unpriced.** The documented failure mode of scheduled headless loops is exactly
   "a headless call fired by a schedule with nobody watching" becoming the expensive one (see Sources).
   Bounds here: 1 unit/tick · 30-min interval · single-flight (lock + live-run skip) · `--max-turns` ·
   watchdog wall-clock · tokens recorded per tick from day one · pause flag Eli can flip from his phone.
   Recommend the first digest to Eli include tick count × avg tokens so the ceiling conversation (strategy
   brief Q1) happens on data.
10. **Double-spawn.** Tick N's session still running when tick N+1 fires (a 35-min triage on a 30-min
    interval). Three independent layers: stamp throttle (guardian), `mkdir` lock with stale-PID check
    (tick script), `live_runs` skip in `loop_precheck()` (DB). Unit-level duplication is separately
    impossible for plans (atomic claim) and prevented for triage/review by single-flight.

## Code Examples

**Guardian hosting block** (append to `momo-guardian.sh`; pattern = the existing nightly-backup block, which
already proves "extra duty riding the 60s reconciler" works [VERIFIED: momo-guardian.sh lines 25–41]):

```bash
# Heartbeat tick (phase 4): throttle to ~30 min via stamp mtime; tick runs detached
# so this reconciler stays single-pass.
TICK_STAMP=/Users/momo/momo/ops/locks/tick.stamp
TICK_INTERVAL=1800
now=$(date +%s); last=$(stat -f %m "$TICK_STAMP" 2>/dev/null || echo 0)
if [ $((now - last)) -ge $TICK_INTERVAL ]; then
  mkdir -p /Users/momo/momo/ops/locks && touch "$TICK_STAMP"
  nohup /Users/momo/momo/ops/momo-tick.sh >> /Users/momo/momo/ops/logs/tick.log 2>&1 &
fi
```
(`stat -f %m` is the macOS/BSD form — GNU `stat -c` will not work here. Source: BSD stat(1).)

**Single-flight lock, macOS-portable (no flock):**

```bash
LOCK=/Users/momo/momo/ops/locks/tick.lock
if ! mkdir "$LOCK" 2>/dev/null; then                    # mkdir is atomic — the POSIX lock idiom
  oldpid=$(cat "$LOCK/pid" 2>/dev/null)
  if [ -n "$oldpid" ] && kill -0 "$oldpid" 2>/dev/null; then exit 0; fi   # live tick — skip
  rm -rf "$LOCK" && mkdir "$LOCK" || exit 0             # stale lock from a crash — reclaim
fi
echo $$ > "$LOCK/pid"; trap 'rm -rf "$LOCK"' EXIT
```

**Launch + watchdog + wrapper telemetry:**

```bash
cd /Users/momo/momo || exit 1
export PATH="/Users/momo/.local/bin:/Users/momo/.bun/bin:/opt/homebrew/bin:/usr/bin:/bin:/usr/sbin:/sbin"

RUN_ID=$(psql -d momo_work -t -A -c "SELECT start_run('mini','tick-${UNIT}',${PROJECT_ID:-NULL},NULL,NULL,'frontier');")

OUT=/Users/momo/momo/ops/logs/tick-out.$$.json
claude -p "Headless tick session. Read /Users/momo/momo/worksystem/heartbeat.md and execute ONE unit: ${UNIT}. Telemetry run id: ${RUN_ID}. You have NO Discord tools — queue Eli-facing output per the runbook. End with one line under 100 chars." \
  --output-format json --max-turns 40 --permission-mode dontAsk --strict-mcp-config \
  > "$OUT" 2>>/Users/momo/momo/ops/logs/tick.err &
CPID=$!; DEADLINE=$(( $(date +%s) + 2700 ))            # 45-min wall clock
while kill -0 $CPID 2>/dev/null; do
  if [ "$(date +%s)" -gt "$DEADLINE" ]; then kill -TERM $CPID; sleep 15; kill -KILL $CPID 2>/dev/null; break; fi
  sleep 20
done
wait $CPID; CODE=$?

read -r TOKENS COST ISERR <<< "$(python3 - "$OUT" <<'PY'
import json,sys
try: d=json.load(open(sys.argv[1]))
except Exception: print("NULL NULL 1"); raise SystemExit
u=d.get("usage") or {}
tok=sum(v for k,v in u.items() if isinstance(v,int) and k.endswith("tokens")) or "NULL"
print(tok, d.get("total_cost_usd","NULL"), 1 if d.get("is_error") else 0)
PY
)"
OUTCOME=ok; [ "$CODE" -ne 0 ] && OUTCOME=error; [ "$CODE" -ge 128 ] && OUTCOME=killed
psql -d momo_work -t -A -c "SELECT finish_run(${RUN_ID},'${OUTCOME}',${TOKENS},NULL,'tick ${UNIT}; exit ${CODE}; cost_json=${COST}');"
```
(usage-key shape inside the JSON is [ASSUMED] — `total_cost_usd`/`session_id`/`result` fields are
documented; exact usage subkeys must be confirmed in the probe run and the parser pinned to reality.
`cost_usd` stays NULL while Fable 5 has no recorded rate — note carries the harness estimate.)

**Migration 012 — pre-check + gate-aware readiness (sketch; DDL runs at the phase gate with Eli's approval,
as every migration does — psql -f always asks):**

```sql
CREATE OR REPLACE FUNCTION loop_precheck() RETURNS TABLE(
  paused BOOL, pending_atoms BIGINT, new_items BIGINT,
  ready_plans BIGINT, seeds_due BIGINT, live_runs BIGINT, stale_runs BIGINT
) AS $$
  SELECT (SELECT value='paused' FROM engine_state WHERE key='loop'),
         (SELECT count(*) FROM atoms WHERE fate='pending' AND decided_at IS NULL
             AND (reasoning IS NULL OR reasoning NOT LIKE 'HELD %')),   -- held atoms wait for Eli, not ticks
         (SELECT count(*) FROM inbox_items WHERE status='new'),
         (SELECT count(*) FROM plans pl JOIN phases ph ON ph.id=pl.phase_id
             JOIN projects pr ON pr.id=pl.project_id
           WHERE pl.status='todo' AND pl.assignee_type='agent'
             AND ph.status<>'blocked' AND ph.stage IN ('planned','executing')
             AND NOT EXISTS (SELECT 1 FROM unnest(pl.depends_on) dep JOIN plans d ON d.id=dep WHERE d.status<>'done')
             AND NOT EXISTS (SELECT 1 FROM plans e WHERE e.phase_id=pl.phase_id AND e.wave<pl.wave AND e.status NOT IN ('done','cancelled'))
             AND NOT EXISTS (SELECT 1 FROM tasks g WHERE g.project_id=pl.project_id AND g.gate IS NOT NULL
                               AND g.status NOT IN ('done','cancelled')
                               AND (g.phase_id IS NULL OR g.phase_id=pl.phase_id))),  -- gate-aware
         seeds_due_count(),
         (SELECT count(*) FROM agent_runs WHERE ended_at IS NULL AND heartbeat_at > now()-interval '10 minutes'),
         (SELECT count(*) FROM agent_runs WHERE ended_at IS NULL AND heartbeat_at <= now()-interval '10 minutes');
$$ LANGUAGE sql STABLE;
```
Apply the same gate condition to `claim_next_plan` (CREATE OR REPLACE in 012) so the claim itself — not just
the pre-check — is gate-safe. Plus: `engine_state` (§5), `notifications(id, created_at, kind, body, run_id,
sent_at)`, `reap_stale_runs()`, grants to `dashboard_ro`.

## State of the Art

| Old approach | Current approach | Changed |
|---|---|---|
| launchd KeepAlive daemons for persistence | cron-triggered single-pass reconcilers (this Mac) | 2026-07-02 incident — launchd pends ALL automatic spawns on this build |
| `-p` loads full interactive context, period | `--bare` exists and "will become the default for `-p` in a future release" | current docs — pin non-bare explicitly NOW so a CLI update can't silently drop the guard |
| Cost tracking by scraping/estimating | `--output-format json` ships `total_cost_usd` + per-model breakdown; `--max-budget-usd` gives a per-invocation ceiling | current docs |
| `/schedule` cloud routines / `claude --bg` background agents | Available in current Claude Code — but they run outside this substrate (cloud, or unsupervised local daemons) | rejected: "no new services"; the guard/telemetry/digest contract requires the tick to own the lifecycle |
| Channels (`--channels`) research preview bound to one interactive session | unchanged | the bridge stays exclusively on the interactive session |

## Assumptions Log

| # | Assumption | Section | Risk if wrong |
|---|---|---|---|
| A1 | Non-bare `claude -p` spawned by cron/guardian loads `.claude/settings.local.json` hooks from cwd (docs say yes; verified live only for work-dir `claude -p` per guardrails.md, not for cron parentage) | §2 | Tick sessions run UNGOVERNED — must be proven via guard.log before first scheduled run |
| A2 | `--strict-mcp-config` (no `--mcp-config`) prevents the Discord plugin's bridge process from spawning in `-p` | Pitfall 3 | Duplicate bridge: Eli's replies break + guardian deadlock — probe with pgrep |
| A3 | OAuth (subscription) resolves in cron-spawned `-p` (keychain accessible in that context) | Pitfall 6 | Every tick errors at auth; loop silently does nothing — probe run catches it |
| A4 | macOS lacks `timeout(1)`/`flock(1)`; `stat -f %m` semantics; python3 present at `/usr/bin/python3` (guard config implies yes) | Pitfalls 7, Code | Script bugs at first run — `command -v` checks in the probe |
| A5 | JSON result usage-subkey names; `is_error`/`num_turns` field names | Code | Wrong token capture — pin parser to the probe's actual output |
| A6 | `--max-budget-usd` functions under subscription auth (docs frame it as "API calls") | Stack | Belt-only loss; `--max-turns` + watchdog remain the hard bounds |
| A7 | Installed claude version supports all cited flags | Stack | Flag rejected at launch — `claude -v` + a dry `-p` in the probe |
| A8 | Guard treats `SELECT start_run(...)` / `SELECT claim_next_plan(...)` as reads — existing runbooks already rely on this | §4 | Already proven interactively by phases 1–3 runs; risk low |

## Open Questions

1. **Does a non-bare `-p` session spawn the Discord bridge, and does `--strict-mcp-config` suppress it?**
   Known: plain interactive `claude` spawns it; docs say strict-mcp with no config loads no MCP servers.
   Unclear: plugin-channel processes specifically. **Recommendation:** plan 04-01 is an empirical probe —
   one manual tick with `pgrep -U momo -f 'plugins/.*discord.*start'` before/after; if the bridge appears
   even under strict-mcp, fallback is plugin suppression via a settings override — decide on the probe's
   evidence, not now. This question **blocks scheduling** (not planning).
2. **Cron-context auth + guard inheritance in one shot** — same probe run answers both (A1, A3): assert
   guard.log ALLOW lines from the tick session + a successful trivial unit.
3. **Interval + bounds constants** (30 min? 40 turns? 45-min wall?) — recommendation: ship the conservative
   defaults above as `engine_state` keys / script constants, and surface them in the first digest so Eli
   tunes them with data (mirrors the seeds-review interval-constants pattern).
4. **Should the tick script nudge the interactive session (`tmux send-keys`) to deliver urgent
   notifications immediately?** Works mechanically, but injected keystrokes can interleave with a live Eli
   conversation. Recommendation: v1 = drain on next interaction + morning digest (success criterion 5 needs
   no overnight DM); park the nudge as a seed.

## Environment Availability

Probes were tool-blocked this session (subagent lockdown). Classification of what the phase needs:

| Dependency | Status | Basis |
|---|---|---|
| cron entry running guardian every 60s | available [VERIFIED: ops/README.md "verified ticking on schedule"] | load-bearing trigger |
| tmux + interactive momo session + bridge | available (live substrate) | notification drain path |
| Postgres 16 `momo_work`, telemetry trio, seeds/gates functions | available [VERIFIED: worksystem SQL read] | — |
| `claude` CLI ≥ version with `-p` json output, strict-mcp, max-turns | assumed-available — **probe: `claude -v`** | missing flags → blocking, resolve by `claude update` (Eli-gated if it counts as install) |
| `/usr/bin/python3` | assumed-available (guard hook runs on it) | JSON parsing |
| `flock`/`timeout` | assumed **missing** — designs avoid them | none needed |
| GROQ/API keys | not needed by this phase | — |

**Missing-blocking:** none known; Open Question 1 is the only potential blocker and has a designed fallback.

## Sources

**Primary (HIGH confidence — read/fetched directly this session):**
- Local substrate: `ops/momo-guard.py`, `ops/momo-guardian.sh`, `ops/momo-loop.sh`, `ops/README.md`,
  `ops/stop-reply-guard.py`, `.claude/settings.local.json`, `worksystem/{003-gsd-schema,005-gates-render,
  008-telemetry,010-ingestion,011-seeds}.sql`, `worksystem/{ingestion,triage,seeds-review,plan-phase}.md`,
  `wiki/ops/{guardrails,work-engine,discord-bridge,session-recovery,gsd-workengine-design}.md`
- Claude Code docs (fetched 2026-07-03): [CLI reference](https://code.claude.com/docs/en/cli-reference),
  [Headless / run programmatically](https://code.claude.com/docs/en/headless),
  [Hooks](https://code.claude.com/docs/en/hooks)

**Secondary (context on comparable builds/failure modes):**
- [Claude Code in CI/CD and Headless Automation](https://hidekazu-konishi.com/entry/claude_code_cicd_and_headless_automation.html)
- [My nightly Claude Code cron was about to start costing real money](https://dev.to/mjmirza/my-nightly-claude-code-cron-was-about-to-start-costing-real-money-2ndm)
- [Prevent cronjobs from overlapping](https://ma.ttias.be/prevent-cronjobs-from-overlapping-in-linux/) ·
  [Cronitor: prevent duplicate cron executions](https://cronitor.io/guides/how-to-prevent-duplicate-cron-executions) ·
  [Postgres advisory locking for scripts](https://www.the-art-of-web.com/sql/advisory-locking/) ·
  [Baeldung: single-instance bash](https://www.baeldung.com/linux/bash-ensure-instance-running)

## Metadata
- Confidence by area: CLI/headless mechanics HIGH (official docs, current) · scheduling HIGH (local
  empirical record) · guard interaction HIGH-for-design / probe-gated-for-cron-parentage · duplicate-bridge
  mitigation MEDIUM (cited, unprobed) · double-spawn/locking HIGH (standard patterns + existing atomic claim)
  · digest/pause design HIGH (substrate read directly)
- Researched: 2026-07-03 · Valid until: ~2026-07-17 (Claude Code CLI moves fast — the `--bare`-default change
  is explicitly announced; re-check headless docs before executing if planning slips)
