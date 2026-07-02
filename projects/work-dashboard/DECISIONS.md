# Decision Log — Work-engine Dashboard

> Autonomous-mode log (Eli's directive 2026-07-02: proceed without per-step gates; log every
> judgment call for after-the-fact review). Format per decision: **Prompt** (the question that
> arose) → **Context** → **Decision** (+ reasoning) → **Implications** → **Priority/Reversibility**.
> If Eli overturns a decision, the refactor gets scheduled and this log updated.

---

## DL-01 · Runtime: Node 24 LTS, not Bun

- **Prompt:** The mini has no Node, only Bun 1.3.14. Run Next.js on Bun, or ask for one install?
- **Context:** Next 16 + Turbopack on Bun has a live trail of verified crash-grade issues (one still open, confirmed-tracked by the Next team); Next's `engines` demands Node ≥20.9. Five later phases (incl. SSE streaming in phase 4) sit on this runtime.
- **Decision:** Node 24 LTS as runtime (Eli approved the install); Bun stays as package manager. The boring supported path for a foundation.
- **Implications:** One extra install on the mini; node@24 is keg-only and NOT linked (`brew link` is guard-blocked) — everything runs with a PATH prefix. If that friction grows, ask Eli to approve a proper link/profile edit.
- **Priority/Reversibility:** High-impact, easily reversible (swap runtime later; `pg` client runs on both).

## DL-02 · DB client: `pg` with hand-typed rows, no ORM

- **Prompt:** pg vs postgres.js vs Drizzle/Kysely for a read-only dashboard?
- **Context:** ~5 read-only SELECTs against views that already exist; the view IS the contract.
- **Decision:** `pg` 8.22 + hand-typed interfaces. ORM codegen earns nothing at this query surface; `pg` is the most battle-tested and Bun-compatible (keeps the fallback path open).
- **Implications:** Row types are pinned by hand (probe-confirmed against the live view); if the query surface grows past ~15 queries, revisit Kysely.
- **Priority/Reversibility:** Medium, easily reversible (lib/queries.ts is the single seam).

## DL-03 · Read-only enforcement: dedicated `dashboard_ro` role, tighter grant form

- **Prompt:** How to make "the app cannot write" true by construction (D-02)?
- **Context:** App-level discipline isn't a guarantee; PG role privileges are. Research offered a blanket function grant vs explicit EXECUTE on just the two read functions.
- **Decision:** Tighter form — SELECT on all tables/views, EXECUTE only on `human_queue`/`gate_blocking`, default-privileges for future views (run as `momo`, the owner), plus `default_transaction_read_only=on` as belt only (privileges are the boundary; the GUC is session-overridable).
- **Implications:** New engine views are auto-readable; new engine *functions* need an explicit grant if the dashboard should call them (deliberate friction). Note: PG grants EXECUTE to PUBLIC by default, so engine functions are technically callable by dashboard_ro — but they all mutate tables, and the missing table privileges stop them (verified: INSERT fails). Write-must-fail is a permanent regression test.
- **Priority/Reversibility:** High (security posture), easily loosened later.

## DL-04 · Port: 3100, not 3000

- **Prompt:** `next start` failed EADDRINUSE — something already serves :3000 and it returned 200.
- **Context:** Investigated before assuming: the occupant is the **BD CRM** Next app already running on the mini (title "BD CRM"). Killing it was never an option.
- **Decision:** Dashboard lives on **:3100** (dev on the same port, `-H 0.0.0.0` for LAN reach).
- **Implications:** Eli's URL is `http://192.168.0.250:3100`. Any future reverse-proxy/consolidation story for the mini's apps is a separate decision.
- **Priority/Reversibility:** Low, trivially changeable (one script line).

## DL-05 · Freshness model: force-dynamic SSR + 10s client refresh

- **Prompt:** How does "refresh reflects real state" work without building phase 4's live streaming early?
- **Context:** pg-backed server components prerender static by default (would ship stale rows forever). Phase 4 needs near-real-time later.
- **Decision:** `export const dynamic = "force-dynamic"` + a 10s `router.refresh()` client interval. Single rendering path (SSR), no client data stores; the `lib/queries.ts` seam is exactly what a phase-4 SSE route reuses.
- **Implications:** Every 10s = one cheap SELECT per open browser tab; fine for one user on localhost. Not a live-stream — that's phase 4's job, path deliberately kept open.
- **Priority/Reversibility:** Medium, upgrade path designed-in.

## DL-06 · Phase 1 skipped the formal UI-SPEC step (methodology deviation)

- **Prompt:** GSD blocks planning of UI phases without a UI-SPEC; the ui-phase machinery isn't built yet. Block phase 1 to build it?
- **Context:** Phase 1's UI is one grid screen on shadcn defaults; phases 2–5 are the properly visual ones.
- **Decision:** Proceed without it for phase 1 only; **build the ui-spec step before planning phase 2**. Logged in the design doc as a deliberate deviation.
- **Implications:** Phase 1's visual polish is baseline shadcn (acceptable for a foundation). The debt is explicit and comes due at phase 2 planning.
- **Priority/Reversibility:** Medium; self-correcting next phase.

## DL-07 · Executor model: main loop executes, subagents research/plan (substrate deviation)

- **Prompt:** GSD runs each plan in a fresh worktree-isolated executor agent. The guard locks subagents to web+read tools — they cannot write files or run builds.
- **Context:** Guard design is deliberate (subagents return data; the main loop persists). Changing the guard is a security decision Eli owns.
- **Decision:** For this build: researcher/planner/checker run as fresh subagents (pure-output roles — full GSD fidelity), execution happens in the main loop, sequentially by wave. Revisit the guard's subagent policy as part of stage 3 (execution loop) design — NOT unilaterally.
- **Implications:** No parallel plan execution within this build (waves serialize). For a solo project on one machine, cost is small. Stage-3 parallel fan-out needs Eli to approve a guard change first.
- **Priority/Reversibility:** Medium now, becomes High at stage 3; fully reversible.

## DL-08 · Gates auto-approve for this project (Eli's directive)

- **Prompt:** Eli (2026-07-02): "keeping me in the loop will actually be more inefficient" — proceed autonomously, decision log instead.
- **Context:** GSD's `--auto` mode exists for exactly this; it's a per-project config flag in our design.
- **Decision:** `project_config.gates.auto = true` for work-dashboard. Remaining phase gates (confirm_plan etc.) auto-complete with a log entry. Hard safety gates (spend/publish/install/delete) are NOT covered — those stay Eli-gated regardless.
- **Implications:** Speed. Risk contained by: adversarial plan-checker still runs every phase, tests still gate every plan, this log captures every fork.
- **Priority/Reversibility:** High-impact process change, instantly reversible (flip the flag).

## DL-09 · Phase 1 verification: self-verified except the cross-machine click

- **Prompt:** Success criterion 1 ("Eli opens it from his MacBook") can't be proven from inside the mini (checker warning).
- **Context:** Everything else verified: LAN bind (200 on 192.168.0.250:3100), counts match DB exactly, route Dynamic, 6/6 tests incl. write-must-fail.
- **Decision:** Phase 1 stage = `verifying`; URL sent to Eli; his first successful load closes it. Not blocking phase 2 on it (risk: macOS firewall prompt — if his load fails, it's a 30-second fix, logged here).
- **Implications:** A residual unknown travels alongside phase 2 work; surfaces naturally the moment he opens the URL.
- **Priority/Reversibility:** Low.

## DL-10 · Phase 2 skipped a dedicated research pass

- **Prompt:** Run the full researcher again for phase 2, or reuse phase 1's?
- **Context:** Phase 2 adds `/projects/[slug]` dynamic routes and a wave-lane layout on the same stack. Phase 1's RESEARCH (valid until 2026-08-01) covers the stack, pitfalls, and freshness model; the schema is ours. GSD treats research as per-phase optional.
- **Decision:** Skip; planner receives phase 1's RESEARCH + the live schema + UI-SPEC. New unknowns (App Router dynamic-route conventions in Next 16) are covered by the scaffold's own bundled docs (`node_modules/next/dist/docs/`) which the executor consults.
- **Implications:** If planning hits a genuine unknown, spin research then (cheap to recover).
- **Priority/Reversibility:** Low.

## DL-11 · Compact ui-phase: spec by orchestrator, checker folded into plan-checker

- **Prompt:** DL-06's debt came due — phase 2 needs a UI-SPEC but the full gsd-ui-researcher/ui-checker machinery isn't adapted.
- **Context:** GSD's ui-researcher is 377 lines and aimed at products needing style exploration. This dashboard's design system is already fixed (shadcn defaults + skill rules + my dataviz discipline); the real value of UI-SPEC here is locking density, status→color mapping, and copy BEFORE code.
- **Decision:** I authored `phases/2/UI-SPEC.md` from GSD's actual template with the design system locked project-wide (notably: a status→color mapping that must never vary between screens, depth ≤2 navigation, density rules). Plan-checker verifies plans against it. Full ui-researcher adaptation deferred until a client-facing build needs style exploration.
- **Implications:** Less exploration than GSD full-fat — appropriate for an internal ops tool; the template structure keeps it upgradeable.
- **Priority/Reversibility:** Medium; the spec is a living doc.

## DL-12 · Phase-1 criterion 1 failed on first try: MacBook can't reach the mini (open)

- **Prompt:** Eli's load of http://192.168.0.250:3100 didn't connect (2026-07-02 ~11:06).
- **Context:** Server verified healthy on that address from the mini itself. Same failure shape as his Parsec 6023/6024 — pointing at the mini's application firewall (headless sessions can't click macOS's "Allow incoming" prompt for node) or network isolation between the machines.
- **Decision:** Diagnostic sent (ping discriminates firewall vs network isolation). NOT treating phase 1 as failed — the app-layer criteria all pass; this is an environment gate I cannot clear from the terminal (guard hard-blocks firewall/system reads and writes, correctly).
- **RESOLVED (same day):** Eli was simply not home — different network, so a local-only app is correctly unreachable (D-01 working as designed). Criterion 1 closes when he's next on the home network. Side discovery from the mini's Parsec host log: his Parsec failures are NAT traversal (-6023/-11010, attempts reaching signaling, "UPNP: No devlist" on the home router). **Refined after Eli pushed back ("it worked remotely before"):** the log shows a successful remote session TODAY 17:23 (direct BUD connection), failures resuming 20:07 — the mini is fully exonerated; the variable is whichever network his MacBook is on (home router has no UPnP so success depends on the client network being hole-punch-friendly). Recommended Tailscale on both machines to make it deterministic (also enables remote dashboard, no public exposure); awaiting Eli's call — if adopted, revisit D-01's remote-access posture (RMT-01).
- **Priority/Reversibility:** Resolved; follow-up owned by Eli (router/Tailscale).

## DL-13 · Engine fix: cancelled dependencies now satisfy dep-gating (checker's catch)

- **Prompt:** The phase-2 plan-checker found the engine split-brained: `claim_next_plan` blocked on any dep `<> 'done'` (a cancelled dep = dependents blocked FOREVER), while wave-gating already treated cancelled as complete.
- **Context:** A cancelled plan is deliberately descoped; updating its dependents is a planning responsibility, not a runtime block. The dashboard's waiting-on logic needed one truth to render.
- **Decision:** Unified engine-side (`worksystem/007-cancelled-dep-semantics.sql`): cancelled satisfies dependencies in BOTH claim functions; the dashboard matches. Alternative (cancelled blocks + distinct rendering) rejected — it makes a descoped plan a permanent roadblock with no engine mechanism to clear it.
- **Implications:** If a plan is cancelled while dependents genuinely still need its output, the planner must re-plan those dependents — that's now an explicit process obligation (worth a checker rule later).
- **Priority/Reversibility:** High (engine semantics), easily reversible.

## DL-14 · PlanRow gained `requirements` mid-plan (planner omission)

- **Prompt:** Phase-2 plan 3 (REQ badges) needs each plan's REQ list, but plan 1's PlanRow query didn't select it and plan 3's file list didn't include lib/queries.ts.
- **Context:** Small contract gap between two plans the checker didn't catch (it verified the phase-row side of coverage, not the plan-row side).
- **Decision:** Added `requirements` to PlanRow + the SELECT during plan 3 execution — 2-line change, logged rather than re-planned. Executor-level deviations of this size get logged in the plan summary + here; anything structural would go back through the planner.
- **Implications:** Test fixtures updated; nothing else touched the seam.
- **Priority/Reversibility:** Low.

## DL-15 · Two execution potholes worth remembering (not decisions, calibration)

- Vitest + Testing Library needs `globals: true` for auto-cleanup — without it, renders leak across tests and getByTestId finds duplicates.
- Badge-style shadcn components carry `aria-invalid:*-destructive` classes in their base string — assert class membership with `classList.contains()`, never substring matching on className.

## DL-16 · Telemetry design (D-04): orchestrator-written lifecycle records, not instrumentation

- **Prompt:** USE-01 needs agent runs recorded — but the agents are Claude processes with no hook to self-report token usage per tool call.
- **Context:** The harness DOES report per-subagent token totals in completion notifications, and the orchestrator (me) sits at every lifecycle moment: spawn, notify, claim, done. The wiki's model-cost-reference has per-model rates.
- **Decision:** Telemetry = two engine tables (`agent_runs` + `agent_events` breadcrumbs), written BY the orchestrator via psql at lifecycle moments: run row on spawn/claim (with host, kind, project/plan/task, model tier), heartbeat + breadcrumb events as work progresses, close with outcome + tokens (from the harness notification) + cost computed at write time from the rate table. A `record_run`/`log_event`/`finish_run` SQL function trio keeps writes one-line. NOT chosen: per-tool-call instrumentation (no hook exists; pretending finer granularity would be fake data) and log-file scraping (fragile).
- **Implications:** Main-loop work (me executing plans directly) is recorded the same way — kind='orchestrator'. Token numbers exist only where the harness reports them; missing = NULL, rendered as absent, never zero. Liveness = heartbeat recency, so a crashed agent shows "stale", not "running" forever.
- **Priority/Reversibility:** High (this is D-04's foundation and phase 5 feeds on it); schema is additive, easily extended.

## DL-17 · Phase-4 execution notes (incl. two DL-16 amendments)

- **Naming amendment (checker's catch):** DL-16 said `record_run`; shipped as `start_run` (clearer verb pairing with `finish_run`). All code, tests, and docs use `start_run` — DL-16 is amended by this entry, no silent divergence.
- **Cost blend rule pinned (checker's catch):** "the rate" = the cost reference's **blended ~3:1 in:out** figure (Sonnet 5 ≈ $4/M, Opus 4.8 ≈ $10/M), documented in the wiki telemetry section. Every cost_usd now has declared provenance.
- **Fable-5 rate gap:** the session's own model (Fable 5) isn't in the cost reference yet, so this build's subagent runs carry exact tokens + NULL cost + explanatory note — invented rates would be fake data. Ask Eli for Fable rates (or add them when published) and backfill is a one-line UPDATE per run.
- **Migration 008 applied under the autonomous directive** (Eli 2026-07-02: "just continue forth"): additive schema, no data deletion, no install, no publish — squarely inside the approved build. Hard-gated categories remain untouched.
- **Criterion-1 scope (checker-flagged, accepted):** "every run records telemetry" is delivered as mechanism + mandatory documented practice + both run shapes exercised live (orchestrator run 2, subagent runs 3-4) — future compliance is procedural (the wiki section the orchestrator reads), not code-enforced. Honest limit on record.

## DL-18 · Phase-5 execution notes

- **Digest timestamps are approximate (checker's catch, accepted + documented):** plan-done and gate-cleared entries use `updated_at`, a last-touch column — a post-completion edit re-dates the entry. Documented in the query; the exact fix (a `completed_at` or status-transition event) is an engine follow-up, not urgent at current volume.
- **Commit/DM digest coverage (planner's honest orphan, remedy adopted):** the telemetry practice now mandates `log_event(id,'commit',…)` and `log_event(id,'dm',…)` breadcrumbs — the digest's agent_event arm picks them up with zero dashboard change. Effective immediately for the orchestrator.
- **Spend agreement is structural:** both spend queries share byte-identical window predicates and SQL owns all sums — the UI cannot show totals that disagree with per-run figures.
- **Live spend state:** every run is currently unpriced (Fable rate gap, DL-17) — the digest shows the honest "N runs unpriced (Fable rates pending)" note. Rates from Eli → one UPDATE per run backfills.

---
*Started 2026-07-02. Updated continuously while autonomous mode is active.*
