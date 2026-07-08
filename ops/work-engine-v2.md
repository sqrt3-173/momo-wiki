# Work-engine v2 — the reliability/enforcement rebuild (2026-07-08)

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
