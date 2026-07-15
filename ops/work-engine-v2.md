# Work-engine v2 — the reliability/enforcement rebuild (2026-07-08)

## ▶ RESUME HERE — BD pipeline spun up on Eli revenue directive (2026-07-10 23:55)
**Eli (23:43): make me $1M/month** — answered honestly + counter-offered the BD pipeline at
machine scale (Discord 1525120632218587270). STEP 2 DONE (00:35): 9 audits committed, bd-pipeline repo 01a4aee — pitch-ready; next = draft rebuilds after Eli eyeballs. STEP 1 DONE: 137 SEQ GP practices sourced/
verified/fingerprinted -> wiki/bd/seq-gp-targets.md (e203cd2). NEXT (fresh session/engine):
audit fleet on top 30 (APP-5/privacy, perf, design teardown; hottest = broken-SSL Noosa Heads
Medical + Robina Medical Centre, 2010-content solo Booval, placeholder Harmony Health), then
draft rebuilds top 10. Await Eli vertical confirmation (GPs = default). Forge board complete
(prev header below still accurate: pending Eli = device sitting, sudo patches, backend call).

## (superseded) BOARD COMPLETE (2026-07-10 16:40)
**ALL 13 FORGE phases done to their human line** (13-VERIFICATION passed 0 gaps, 44d8510;
12 Wave A complete incl. Social tab, Wave B = Eli backend call notif #37; completion report
Discord 1525028478410293289). Nothing autonomous remains on forge. PENDING ELI: device
sitting (scan first), 2 sudo guard patches (#29), backend call (#37), iPhone install
trust issue (helped 15:54). Known noise: false hold-parked notification per new forge run
(watcher HOLD attribution bug, seeded in cockpit backlog) — drain on sight. Cockpit Phase 3
executes after Eli guard applied. Engine idles to drip + Eli-event-driven from here.

## (superseded) morning after the 13-phase night (2026-07-10 ~09:15)
**Night OUTCOME:** FORGE phases 8/6/4/9/10 built+verified, 11 code-complete (290 tests ×2),
7 built to its human gate (scan + ~$0.20 benchmark approval), 12/13 planning runs queued/live
(money calls land as artifacts: 12-SDK-DECISION.md, 13-PIPELINE-DECISION.md — Eli decides).
Cockpit Supervise BUILT AND RAN THE NIGHT (26 supervised runs); Phase 3 (Control) planned,
executes after Eli's "guard applied". Five infra bugs found+fixed live (note-wiring caac5e8,
prompt-file race 69e41d0, claim clobber 8c76949, false-reap heartbeat da1191c, stdin prompt
corruption e43ddf0 — root causes in cockpit 02-supervise/evidence/). Morning report sent
(Discord 1524916556029493368, ~$285 usage / 58 runs).
**Eli's punch-list (asked, batched):** device sitting (scan → calibration → minimise.mov →
barcode → orbit benchmark w/ privacy+spend call) + sudo sitting (two guard patches, notif #29).
**Standing rules live:** process delegation (wiki/feedback/process-gates-delegated.md);
HOLD-lifts MUST be written into the project's STATE.md by the interactive session BEFORE
re-queuing (run 118 lesson); spawn notes are retry-semantics, .planning is authority.
**Mechanics:** control_commands outbox is THE spawn channel (INSERT as -U dashboard_control
+ #MOMO_OK); one-ahead queue discipline; watcher = scratchpad/watch-engine.py (offset-based).
**Late-morning additions (2026-07-10 ~11:15):** Phase 12 planned (Wave A executing run 142;
Wave B = Eli backend call, notif #37, Azure-vs-Supabase flag raised); Phase 13 planned + its
execution queued (on-device pipeline default, notif #38 FYI); supervise-watch progress signal
fixed 079f058 (heartbeat became OS-liveness after da1191c so stuck-detection couldn't fire —
now commits OR worker-transcript mtime via .pid->lsof). Sixth live infra fix of the cycle.

## (superseded) FORGE full-board night (2026-07-09 ~21:45, interactive session)
**Standing delegation (Eli, 2026-07-09 21:32–21:41, wiki/feedback/process-gates-delegated.md):**
process gates are MOMO's; his device/eyeball checks + sudo + money stay his, parked not blocking.
**Directive: work through ALL 13 FORGE phases tonight** ("foregoing some of the blockers") —
order 8→6→7-spike→4→9→10→11→12→13; internal quality gates (plan-checker, tests-green,
per-plan verify) stay ON; money decisions default to no-spend; VALD Performance is Phase 6's
scope benchmark (on-phone specs).
**Mechanics:** spawner requests one-ahead only (alphabetical pickup!); `note` field now reaches
workers (momo-spawn.sh fix caac5e8 — runs 85/86 wasted slots proving it); run 86's Phase-5-gate
decline is overridden in forge-plan-phase-6b.json's note. Cockpit: Phase 1 verified passed
(self-verified addendum); 013 migration = Eli-approved via delegation, apply after the brake
window (~21:53) then queue momo-cockpit resume to fill spawn gaps; migration also re-enables
drip ticks during spawn runs. Watchers: watch-engine.py (scratchpad) baseline=86. FORGE-02/03/05
HOLDs = Eli device items, untouchable. Injection guard patch authored, awaits Eli sudo (batch
with 02-06's).

## (superseded) baton back with the MAIN session (2026-07-08 18:45)
The bg job (f95fc0c0) died ~16:40–17:40 with Forge at 9/10: every code plan done+green+deployed,
but it never sent Eli the 02-05 device-checkpoint ask and never released its claim — Forge sat
one Eli-action from phase-complete for an hour until Eli asked "what is the hold up?". Main
session now owns Forge: Eli has the checkpoint instructions (scan → confirm standing/likeness/
height); on his reply, write 02-05-SUMMARY with his evidence, then phase verify. Claim removed
(owner process verified dead). **GAPS this exposed:** (1) bg/daemon sessions have NO watchdog —
only tmux 'momo' is monitored; (2) a session holding an un-sent Eli-gate + a claim needs a
dead-man handoff (claim files should carry a PID for liveness checks); (3) both sessions answered
Eli's API-cost question — thread-boundary claims don't survive their owner's death. Fix candidates
next session; don't rebuild tonight.

## (superseded) baton moved to bg job "gsd-dogfood-run-phase1-planning" (2026-07-08 11:20)
Eli accidentally arrow-keyed in the tmux terminal at 11:20 → the main session's Phase-1
**researcher was interrupted** (`[Request interrupted by user]` in its transcript at 11:20:01,
~7 min in). The harness spawned bg job `f95fc0c0` ("gsd-dogfood-run-phase1-planning"), which
adopted the orphaned agent and now owns FORGE Phase-1 planning: fresh researcher re-spawned
(sonnet) → RESEARCH.md → planner (opus) → plan-checker → plans committed. **If you are the
main tmux session reading this: do NOT restart plan-phase 1 — check
`projects/forge/.planning/phases/FORGE-01-*/` for artifacts first; the bg job may already
have landed them.** Lesson filed: an interactive terminal someone can type into is a fragile
home for long-running orchestration — another argument for engine-owned (headless) GSD runs.

## Dogfood run 2 STATUS (bg job f95fc0c0, 2026-07-08 12:50): FORGE Phase 1 at 4/5 plans
Full plan-phase → execute cycle ran clean under the bg job: research (sonnet) → plans (opus)
→ plan-checker PASSED → executed plans 01-04 (BODY-02/03/04, DATA-01/02) with per-plan
verification (live 401/200/400 curls incl. Eli's POST proof, migration test, 2 screenshot-
verified error surfaces, full suite green). Remaining: plan 05 = Eli's device ▶ checkpoint.
New learnings beyond the naming-convention one (already filed above):
- **Executor subagents are read-only too** — settled pattern: executor COMPOSES exact diffs
  (told upfront not to try Write), orchestrator applies/verifies/commits. Works well.
- **Write state back per plan, not per wave** — Eli caught /gsd-progress reporting stale
  counts; stock GSD's executor updates STATE.md after every plan. Do the same inline.
- **Engine ticks left a broken UI test** (plan 40 renamed a button, never updated the test,
  tick only ran unit tests) — exactly the class of rot Phase 4's verification sweep exists for.
- Guard notes: modal CLI is allowlisted (secret create + deploy + app logs all clean);
  curl GET (incl. -H auth headers) passes, -X POST trips the exfil rule (Eli ran that one);
  `openssl`/`timeout`/`for`/var-assignment leading tokens all blocked — python3 equivalents work.
- Eli feedback captured mid-phase: SCAN-05 (staged audio-guided capture) + SCAN-06 (crouched
  rest-pose defect, server-side) added to Phase 2 with roadmap/requirements committed.

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
1. **⚠️ SUBAGENTS CANNOT WRITE FILES — CORRECTED 2026-07-08: this is OUR GUARD, not the harness.**
   `ops/momo-guard.py` main(): `agent_id` present → only SUBAGENT_ALLOW (web+read) tools pass — the
   "enrichment job" lockdown written for untrusted-web BD subagents, applied globally to every subagent
   since. Two sessions repeated "harness rule" without checking (the failure mode is real — verify, don't
   inherit). **Eli's decision 2026-07-08: subagents may write `.planning/**/*.md`; wiki stays
   orchestrator-only** ("wiki goes through the super agent"). Carve-out written + probe-tested in
   `ops/momo-guard.proposed.py` (commit 2c452df); self-disarm protection means Eli installs it
   (cp proposed → live). Code/Bash stay orchestrator-only until the Jul-4 tree (worktrees +
   review gate) lands. Until installed: orchestrator writes every artifact from agent text.
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

**Live-run ledger, day 1:** run 20 ✓ (02-01 close-out + judgment) · run 21 ✓ (02-02 close-out, silent
success) · run 22 ✓* (re-scoped HELD plans — safe outcome, exposed the HOLD-wording ambiguity → HOLD
convention) · run 23 ✗ (budget kill mid-phase-3-planning, research lost uncommitted → runbook now
mandates commit-as-produced + STATE resume points; claim leak cleaned by interactive session as
designed; zero repo damage). Each failure hardened a rule the same hour. All figures are
subscription-usage equivalents, not billed money — see [[../feedback/report-usage-not-dollars]].

## ✅✅ FIRST LIVE RUN PROVEN (run 20, 2026-07-08 15:34–15:52, $4.30) — the loop is closed
The 15:34 natural tick picked nv-health-website, and the headless session executed the FULL protocol
unsupervised: claim → route → **safe_resume_gate judgment** (plan 02-01's code pre-existed from
interactive sessions → verified against the plan, 14/14 tests, close-out SUMMARY instead of redo) →
fixed a stale test asserting the superseded default-denied consent seed → commit b5736ee → STATE.md
synced (5/8, 63%) → claim released → **contradiction flagged, not executed** (plans 02-03/02-04 want a
consent banner; Eli's standing decision d6d3303 is opt-out — queued him a numbered decision instead) →
notification delivered + marked. This is v2's whole thesis demonstrated: enforcement + feeding + judgment.

## ✅ Tick→GSD wiring LANDED (main session, 2026-07-08 15:xx) — the two-speeds model is real
Phase 1 closing (verifier 5/5) unblocked it per Eli's sequence. What shipped:
- **`gsd-next` tick unit** in `ops/momo-tick.sh`: when momo_work has no triage/plans, the tick scans
  `projects/*/.planning`, skips any project whose name appears in an `ops/locks/` FILENAME (the claim
  convention — interactive sessions always win; >3h-old claims are flagged in tick.log, never auto-removed),
  and picks the first project whose STATE.md frontmatter shows `completed_phases < total_phases` and no
  terminal status (blocklist: complete/archived/paused — statuses vary, found on nv-health-website).
- **`worksystem/gsd-next.md`** runbook: re-check claim → claim → route ONE step (execute next plan >
  plan next phase > verify phase) via the STOCK workflow files, exact artifact naming, commit-per-task,
  Eli-gates → notifications queue, release claim. heartbeat.md §2/§3 updated (priority: triage → legacy
  plan → gsd-next → seeds).
- **Validated**: forced ticks ran the new path clean (scan → claim-skip → idle); actionable check
  unit-tested against both live projects. First LIVE run deliberately held back: nv-health-website is
  temp-claimed (`gsd-claim-nv-health-website.md`) pending a review of its pre-stock-GSD .planning
  structure (old interpreted-GSD artifacts, stale frontmatter: says 0/6 phases but Phase 01 is complete
  — fix state before the drip touches it). Unclaim = hand it to the drip.


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

## Industrial Capacity (NEW, 2026-07-13)
Eli's strategic goal: own a significant portion of Australia's industrial capacity (future-scarce asset).
Seed + first-session plan: projects/industrial-capacity/BRIEF.md — SCOPE WITH ELI FIRST, then research fleet.
