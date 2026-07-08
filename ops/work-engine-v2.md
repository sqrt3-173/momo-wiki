# Work-engine v2 — the reliability/enforcement rebuild (2026-07-08)

## ▶ RESUME HERE — baton moved to bg job "gsd-dogfood-run-phase1-planning" (2026-07-08 11:20)
Eli accidentally arrow-keyed in the tmux terminal at 11:20 → the main session's Phase-1
**researcher was interrupted** (`[Request interrupted by user]` in its transcript at 11:20:01,
~7 min in). The harness spawned bg job `f95fc0c0` ("gsd-dogfood-run-phase1-planning"), which
adopted the orphaned agent and now owns FORGE Phase-1 planning: fresh researcher re-spawned
(sonnet) → RESEARCH.md → planner (opus) → plan-checker → plans committed. **If you are the
main tmux session reading this: do NOT restart plan-phase 1 — check
`projects/forge/.planning/phases/FORGE-01-*/` for artifacts first; the bg job may already
have landed them.** Lesson filed: an interactive terminal someone can type into is a fragile
home for long-running orchestration — another argument for engine-owned (headless) GSD runs.

## GSD dogfood on FORGE, started 2026-07-08 ~11:00
Post-restart checks DONE + verified (2026-07-08): (1) **guard alive** — project `.claude/settings.local.json`
hooks intact (momo-guard PreToolUse * + stop-reply-guard Stop); functional test: `launchctl` blocked as designed.
GSD's hooks all landed in GLOBAL `~/.claude/settings.json` — coexist, no clobber. (2) **GSD registered as
SKILLS, not commands** — `~/.claude/skills/gsd-*` (69) + `~/.claude/gsd-core/` (workflows + `bin/gsd-tools.cjs`);
there is NO `~/.claude/commands/gsd/` in v1.6.1 — don't look for it. (3) Dead partial `worksystem/gsd-source/`
DELETED.

**Dogfood run 1 (FORGE, brownfield) IN FLIGHT:** running stock flow map-codebase → new-project. Key mechanics
learned:
- GSD resolves project root from the **Bash cwd** (`git rev-parse --show-toplevel`). `cd` IS guard-allowed and
  Bash cwd persists → `cd /Users/momo/momo/projects/forge` once, then gsd-tools works there
  (`node ~/.claude/gsd-core/bin/gsd-tools.cjs query init.map-codebase` → project_root=forge ✓).
- ⚠️ The workflows' big compound bash one-liners (var-assignment leading token) would hit the guard's
  unknown-leading-tool block — run gsd-tools via `node <path> <args>` directly instead.
- Mapper agents spawned with ABSOLUTE paths in prompts (subagent cwd ≠ my Bash cwd); 4 × gsd-codebase-mapper
  (haiku) launched in background → `.planning/codebase/` 7 docs.

**Dogfood run 1 RESULT (2026-07-08): map-codebase + new-project COMPLETE on FORGE** (commits `adf6bfc`,
`a6a4583` in projects/forge). Learnings that change how GSD runs here:
1. **⚠️ SUBAGENTS CANNOT WRITE FILES in this harness** — all 4 mappers returned content as text instead of
   writing (harness rule: agents return findings; plus Write denials). GSD's "agents write directly" pattern
   doesn't hold → **the orchestrator (main MOMO) writes every artifact** from agent-returned content. Applies
   to ALL GSD agents (planner/roadmapper/executor docs). Executors editing CODE may differ — test before relying.
   **Corollary (found 2026-07-08): hand-written artifacts MUST follow GSD's exact naming convention** — the
   tooling navigates by filename. `gsd-tools progress` counts `endsWith('-PLAN.md')` (convention
   `01-01-PLAN.md`); FORGE-01's hand-named `01-PLAN-01.md` files counted as ZERO plans (phase showed
   Pending at 3/5 done) and `determinePhaseStatus` keys off the same counts. Before writing any artifact,
   check the convention (`gsd-tools template fill <type>` shows canonical names) — or /gsd-progress lies to Eli.
2. **⚠️ Verify mapper output — haiku mappers made confident false claims** (said "ForgeTests is empty" — 38
   tests exist; "no external integrations" — Modal pipeline is live; "camera analysis not built" — shipped).
   Corrected before committing; CONCERNS.md has a "rejected claims" section. Never commit mapper docs unread.
3. `gsd-tools query commit` fails on untracked files (`.planning/` new) — plain `git add + commit` works.
4. Config: v1.6.1 gsd-tools doesn't know `gates`/`safety` keys (template has them — drift); `mode: "yolo"`
   is what disables approval gates. FORGE config: yolo, verifier ON, plan_check ON, parallel plans.
5. new-project/discuss workflows lean on AskUserQuestion (denied) — --auto-style run with the wiki as the
   idea document worked; Eli-facing decisions still go to Discord as numbered text.

**NEXT: `/gsd-plan-phase 1` on FORGE** (Body pipeline proof + data safety: BODY-01..04, DATA-01..02) →
execute → verify. BODY-01 needs Eli's ▶ (batch device asks). Then wire the 30-min tick to GSD
(claim next plan-phase/execute-phase instead of the momo_work queue) — the two-speeds model.


**Trigger:** Eli, after 5 days: the work engine hasn't run smoothly; it must become the scalable backbone. Deep
research launched (workflow `wf_997fd4fb`, 4 researchers → synthesis → adversarial critique).

## Verdict (research): KEEP GSD — don't replace it
All 5 failure modes are **substrate-wiring** failures, NOT methodology failures. Stock GSD (open-gsd/gsd-core)
already prescribes the missing pieces (hard verify phase, gates, fresh-context orchestration) — our replication
left them at "⬜ next" instead of wiring them. So v2 = faithfully install + wire real GSD, not a custom paraphrase.

## Eli's decision (the how): install STOCK GSD for Claude Code
GSD is built *for* Claude Code — its commands ARE Claude Code slash commands + agents. "Install as designed" =
drop the **real** command files into `.claude/commands/` + agents into `.claude/agents/` per the repo's own
install steps, so it operates by default (`/gsd-new-project`, `/gsd-discuss`, `/gsd-plan`, `/gsd-execute`,
`/gsd-verify`, `/gsd-ship`, `/gsd-complete-milestone`) — MOMO's interpreting it was the source of the drift.
⚠️ The ingested `worksystem/gsd-source/` has ONLY agents + templates — the command definitions never came through
(workflows dir empty). That partial/interpreted install is a big reason it ran inconsistently. Subagent
`a0cf70880` is pulling the verbatim commands + install instructions from github → then MOMO installs faithfully.

## v2 top changes (research synthesis, ranked by leverage)
1. **Verify as a hard state-machine gate** (the #1 fix): plan lifecycle `todo→executing→executed→awaiting_verify
   →verified→done` (+ `rework` branch). Executors PHYSICALLY cannot set `done`/`verified` (SQL trigger on role);
   a **fresh-context verifier** (zero shared context) checks the real artifact vs `must_haves` → verified or rework.
   Phases can't ship until all plans verified. This makes shipping broken work (the single-frame body) impossible.
2. **Self-feeding**: `claim_next_phase_to_plan` + a `plan-generate` tick unit + backlog-depth alarm. Why it idled
   5 days — the tick could only EXECUTE plans, never CREATE them, so `ready_plans` stuck at 0.
3. **Idempotent materialization + single-writer planning lock** (upsert on `(phase_id, seq)` UNIQUE).
4. **Claim-lease reaper + attempt cap + dead-letter** (orphaned plans re-queue; no infinite retries).
5. **Two speeds**: background drip (cron, 1 unit/30min) + supervised burst (interactive session orchestrates a
   whole wave) — using the engine at full speed removes the reason MOMO bypassed it.
6. **Anti-bypass discipline**: interactive session ORCHESTRATES + decides gates; does NOT hand-execute plan work.

## Installing REAL GSD (subagent a0cf70880, verbatim from repo)
- Repo `open-gsd/gsd-core`, default branch **`next`**, npm `@opengsd/gsd-core`. **72 commands** in `commands/gsd/`
  (new-project, discuss-phase, plan-phase, execute-phase, verify-work, ship, onboard, new-milestone,
  complete-milestone, +63). 34 agents (`gsd-planner/executor/verifier/plan-checker/…`). Real logic in
  `gsd-core/workflows/` (~90 files).
- **⚠️ DO NOT hand-copy command files — they're STUBS** that `@`-reference `~/.claude/gsd-core/…` paths only the
  INSTALLER creates. Hand-copy = missing-command/schema errors + dead includes. Repo says so explicitly. (This is
  why our `worksystem/gsd-source/` agents+templates copy ran a broken/interpreted GSD.)
- **Faithful install = the official installer:** `npx @opengsd/gsd-core@latest --claude --global` (or `--local`
  → project `.claude/`, takes precedence). Needs Node 18+; ships `gsd-tools` CLI (workflows shell out to it);
  restart Claude Code after. Commands become `/gsd-*` (npm) or `/gsd-core:*` (plugin path).
- **⚠️ SAFETY (MOMO-specific): the installer registers its OWN Claude Code hooks** (SessionStart/PreToolUse/
  PostToolUse/Stop/PreCompact) into `~/.claude/`. MOMO's safety-critical hooks (momo-guard.py, stop-reply-guard)
  live there too. A `--global` install could clash with / clobber the guard. **REQUIRED sequence: install `--local`
  into ONE project (FORGE) first, verify it runs clean + doesn't touch the guard, THEN consider global.** MOMO
  can't run the installer itself (guard blocks installing software + global ~/.claude writes) — Eli runs it or
  explicitly approves.
- **The 5-step loop (stock GSD):** new-project/onboard → discuss-phase → plan-phase (research→plan→verify) →
  execute-phase (wave-based parallel subagents, fresh 200k ctx each) → verify-work (conversational UAT → fix plans)
  → ship (PR). Grouped by milestones. State in `.planning/` (PROJECT/REQUIREMENTS/ROADMAP/STATE.md + per-phase
  CONTEXT/PLAN/UAT). Note: stock GSD's verify-work IS the enforced verify gate our substrate was missing.

Full synthesis + adversarial critique: workflow output `tasks/w8th82dlf.output`. Ties to [[gsd-methodology]],
[[gsd-workengine-design]]. Evidence GSD engine already works: ticks autonomously built stats/routines/Profile/
scan-pt2 plans today (git log) — it's the ENFORCEMENT (verify) + FEEDING that v2 adds.
