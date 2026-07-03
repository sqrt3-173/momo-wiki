# Phase 3 Research: Seeds & Weekly Synthesis

**Project:** ingestion-loop · **Phase:** 3 — Seeds & Weekly Synthesis
**Researched:** 2026-07-03
**Domain:** Internal/architectural — idea-lifecycle store, periodic synthesis review, decision-shaped reporting. No UI, no new services, no package installs.
**Overall confidence:** HIGH on substrate integration (every extension point read directly this session); MEDIUM on external methodology patterns (well-sourced, but qualitative domains).

---

## User Constraints

**Locked decisions (from orchestrator — never recommend against):**
- Substrate: the existing engine (momo_work Postgres, wiki, Discord via reply tool, PreToolUse guard, telemetry, dashboard).
- Originals immutable — inbox rows never edited or deleted (append-only).
- Triage is REASONING against the goals layer — no pattern-rules engine (Eli's explicit call).
- Loop ticks must be near-free when idle: shell pre-check before any Claude session wakes.
- The guard governs spawned sessions exactly as it governs the main loop.
- Stack conventions: Postgres 16 (local, momo_work), plain SQL migration files (`worksystem/NNN-*.sql`), agent prompts as markdown in `worksystem/agents/`, runbooks as markdown, wiki pages for anything Eli reads. No new services, no external SaaS, no spend without Eli's approval.

**Locked substrate fact:** seeds currently live as rows in `momo_work.atoms` with `fate='seed'` and `fate_ref='review:YYYY-MM-DD'` (machine-parseable). The triage runbook explicitly defers the durable seed store to this phase: "The seeds TABLE is phase 3: do NOT build it" [VERIFIED: /Users/momo/momo/worksystem/triage.md step 4, read this session].

**Claude's discretion:** everything else — seed-table shape, lifecycle states, review scheduling policy, clustering method, report format, wiki surface.

**Deferred ideas:** none passed.

---

## Phase Requirements

| REQ-ID | Description | Findings that enable it |
|--------|-------------|------------------------|
| SEED-01 | Seeds carry review dates and full provenance | Seeds-table design (Architecture Patterns §1): `review_on NOT NULL` + `atom_id` FK → `atoms.item_id` → immutable inbox row; migration of existing `review:` rows (Runtime State Inventory) |
| SEED-02 | Weekly synthesis clusters all live seeds + recent knowledge atoms; critical-mass clusters get pre-work; shaped numbered report incl. change-sets | One-pass LLM-reasoning clustering (Architecture Patterns §2, Don't Hand-Roll); pre-work bound pattern from goal-drill; report reuses phase-2 fate procedures (proposals page, PLAN-CHANGES.md); report-design findings (Pitfalls §1) |
| SEED-03 | Every retirement carries a written reason; nothing disappears silently | Status CHECK constraint forcing `status_reason` on every non-live status (by-construction, mirroring phase 1's immutability ethos); no-DELETE trigger; review-report pages as durable narrative |

---

## Architectural Responsibility Map

The engine has no browser tier in this phase; the map uses the engine's actual tiers (the plan-checker should verify against these):

| Capability | Owning tier | Rationale |
|---|---|---|
| Seed store, lifecycle states, review-date scheduling, due-check | **Database (momo_work)** — new `worksystem/011-seeds.sql` | Durable state with constraints lives in SQL, like 010-ingestion; phase 4's shell pre-check must be able to answer "seeds due?" with one cheap query, so due-ness must be a DB fact, not markdown parsing |
| Clustering + synthesis judgment | **Toolless prompt-only subagent** (`worksystem/agents/seed-synthesizer.md`, run via the Agent tool) | Same proven pattern as triage.md and goal-driller.md — pure text→JSON, no DB, no tools; reasoning stays inspectable and re-runnable on malformed output |
| Pre-work (research, fit-vs-existing-systems, mini-brief) | **Orchestrator + bounded research subagents** | Mirrors the goal-drill flow's hard bound ("at most 2 research agents per drill") [VERIFIED: triage.md Goal-drill flow §4] |
| Gather → execute → record procedure | **Runbook markdown** (`worksystem/seeds-review.md`) | Convention: exact commands, zero interpretation (ingestion.md, triage.md precedent) |
| Eli-visible seed surface + review reports | **Wiki** (rendered seedbed page + per-review report page) | Anything Eli reads is a wiki page (locked convention); ROADMAP.md's "rendered from momo_work — do not hand-edit" pattern already exists [VERIFIED: wiki/projects/ingestion-loop/ROADMAP.md header] |
| The numbered decision DM | **Discord reply tool** | HARD RULE: every message to Eli via reply tool, plain numbered text; persist-first-then-DM (triage.md project-fate precedent) |
| Executing Eli's start/merge-park/kill answers | **Orchestrator, reusing phase-2 fate procedures** | "start" → the existing proposals-page + Eli's-gate procedure; "merge into existing project" → the existing PLAN-CHANGES.md lane; no new execution machinery needed |

---

## Summary

This phase is a **schema + agent-prompt + runbook** phase on a substrate that already works — no packages, no services, no UI. The three genuinely new pieces are: (1) a durable `seeds` table that promotes seed state out of the overloaded `atoms.fate_ref` string into real columns with by-construction guarantees (`review_on NOT NULL` for SEED-01; a CHECK forcing a written reason on every non-live status for SEED-03; a no-DELETE trigger mirroring the inbox pattern); (2) a **one-pass LLM-reasoning synthesis agent** — for a corpus this small (seeds will number in the tens, not thousands), an embeddings pipeline is strictly worse: more moving parts, no reasoning trace, and the research consensus is that per-document LLM reasoning is only "costly and impractical" at large scale, which this is not; (3) a review runbook that ends in a persist-first wiki report plus ONE numbered Discord DM, with Eli's answers executed through the fate procedures phase 2 already proved.

The external research converges on one theme: **parked-idea systems die of two diseases — hoarding without processing (the collector's fallacy / backlog graveyard) and review rituals that re-list instead of deciding (review fatigue)**. Every comparable discipline (GTD someday/maybe, Zettelkasten inbox, backlog grooming) lands on the same prescriptions: review on a schedule, prune ruthlessly with recorded reasons, keep the review output decision-shaped and short, and use expanding intervals so dormant items don't nag weekly. Spaced-repetition scheduling (7→14→30→60→90-day re-park intervals, with an escalation to a forced keep-or-kill after ~3 re-parks) imports directly as the review-date policy.

The migration surface is small but real: existing seed atoms encode their review date in `fate_ref='review:YYYY-MM-DD'`; this phase must backfill them into the new table and update `fate_ref` to `seed:<id>` (the format `atoms.fate_ref`'s own schema comment already anticipates: "wiki path / project slug / task id / **seed id**") [VERIFIED: worksystem/010-ingestion.sql line 53]. The triage runbook's seed procedure changes from date-string encoding to an INSERT into the new table.

**Primary recommendation:** Build `worksystem/011-seeds.sql` (seeds table + seed_reviews log + no-DELETE trigger + due-check function), `worksystem/agents/seed-synthesizer.md` (toolless one-pass clustering agent, triage-style JSON contract), and `worksystem/seeds-review.md` (runbook: gather → synthesize → bounded pre-work → persist report → render seedbed → one numbered DM → execute answers via phase-2 procedures), plus a rendered `wiki/seeds/SEEDBED.md` linked from Home — in that dependency order.

---

## Standard Stack

No installs. Everything is already on the machine and proven by phases 1–2.

### Core

| Component | Version/location | Status | Role |
|---|---|---|---|
| Postgres (`momo_work`) | 16, local | [VERIFIED: locked constraint; phases 1–2 verified against it per ROADMAP progress table] | Seed store, lifecycle, due-check |
| Agent tool (toolless prompt-only subagents) | built-in | [VERIFIED: triage + goal-driller run this way; phase 2 stage=verified] | Synthesis clustering, pre-work research |
| Discord reply tool (plugin:discord) | installed | [VERIFIED: MCP server present this session] | The numbered weekly report DM |
| Wiki (git-synced Obsidian vault) | /Users/momo/momo/wiki | [VERIFIED: read this session] | Seedbed page, review reports |

### Supporting
- PreToolUse guard approval-marker convention for engine psql writes [VERIFIED: triage.md step 5 "as with all engine writes"].
- Telemetry trio `start_run/log_event/finish_run` — mandatory for the review run [VERIFIED: wiki/ops/work-engine.md Telemetry section].

### Alternatives considered

| Option | Verdict |
|---|---|
| Embeddings pipeline (pgvector / sentence-transformers) for clustering | **Rejected** — see Don't Hand-Roll; corpus too small, adds an install + a pipeline with no reasoning trace |
| Seeds as a wiki page as source of truth (no table) | **Rejected** — phase 4's shell pre-check needs a cheap SQL answer to "seeds due?"; markdown parsing in a pre-check violates the near-free-idle constraint |
| Keep seeds encoded in `atoms.fate_ref` strings | **Rejected** — no NOT NULL review date (SEED-01 unenforceable), no lifecycle states, no reason column (SEED-03 unenforceable), no place for `times_reviewed` escalation |

**Install command:** none.

---

## Package Legitimacy Audit

Not applicable — this phase installs zero packages. (Flag for the plan-checker: any plan that introduces an install is out of scope for this phase and violates the no-new-services constraint.)

---

## Architecture Patterns

### Data flow (primary use case traceable input → output)

```
triage decides fate='seed'
   └─> INSERT seeds(atom_id, summary, review_on, …)  ← replaces the 'review:' fate_ref encoding
         └─> atoms.fate_ref = 'seed:<id>'  (provenance: seed → atom → immutable inbox item)

[weekly trigger: manual "run the seeds review" now; phase-4 tick's "seeds due?" later]
   └─> GATHER: all status='live' seeds
              + knowledge atoms decided since the last review (fate='wiki', decided_at > last run)
              + projects/phases index + goals pages          (one query each — never per-seed)
   └─> SYNTHESIZE: seed-synthesizer agent (toolless, ONE pass over the whole corpus)
              → JSON: clusters (member seed ids, theme, critical_mass verdict, argued reasoning),
                singletons (re-park w/ next interval | recommend-kill w/ reason), questions
   └─> PRE-WORK (critical-mass clusters only, HARD BOUND ≤2 research agents per review):
              research gaps · fit-vs-existing-systems (against the projects index) · mini-brief
   └─> PERSIST FIRST: wiki/seeds/reviews/YYYY-MM-DD.md (full report, provenance links)
              + re-render wiki/seeds/SEEDBED.md (live seeds, review dates, sources)
              + INSERT seed_reviews row (run receipt) · commit + push wiki
   └─> ONE numbered Discord DM via reply tool: start / merge-park / kill per cluster,
              change-sets against existing projects, proposed retirements WITH reasons
   └─> ELI ANSWERS → execute via existing phase-2 procedures:
              start      → wiki/projects/proposals/<title>.md + Eli's project gate (never auto-create)
              merge-park → seeds.status='merged' (reason + surviving cluster ref)
                           or PLAN-CHANGES.md entry for an existing project (existing lane)
              kill       → seeds.status='killed' + status_reason (Eli's words cited)
              re-park    → review_on = today + next expanding interval, times_reviewed += 1
```

### Pattern 1 — Seed lifecycle as a constrained status column, not an event log

Small, mostly-one-way state machine: `live → merged | promoted | killed` (re-park mutates `review_on`, not status). Enforce SEED-03 **by construction**: `CHECK (status = 'live' OR status_reason IS NOT NULL)` — a retirement without a written reason is un-INSERT-able, the same philosophy as phase 1's immutability triggers. Add a no-DELETE trigger (copy the `inbox_items_guard_delete` shape) so nothing disappears silently even by accident. Full audit-shadow-table/event-sourcing machinery (the standard Postgres audit-trigger pattern [CITED: https://wiki.postgresql.org/wiki/Audit_trigger, https://supabase.com/blog/postgres-audit]) is **overkill for four states on tens of rows** — the review-report wiki pages already provide the narrative history with receipts. Don't build it.

### Pattern 2 — One-pass LLM-reasoning clustering (affinity mapping in a single prompt)

The synthesis agent receives the ENTIRE live corpus in one prompt and returns clusters with argued reasoning — structurally identical to LLM-assisted thematic analysis / affinity mapping: task description, grouping guidelines, required output of groups **with reasons for grouping** [CITED: https://arxiv.org/html/2511.14528v1]. This satisfies the "synthesis pass, never per-seed walk" criterion literally: one agent call, whole corpus. Recent knowledge atoms ride along as **context, not decidable items** — they let the agent argue "cluster X reached critical mass because we recently learned Y"; decisions in the output apply to seeds/clusters only.

### Pattern 3 — Expanding-interval re-parking with forced escalation

Review dates follow spaced-repetition-style expanding intervals — each re-park pushes the next look further out (7 → 14 → 30 → 60 → 90-day cap) [CITED: https://en.wikipedia.org/wiki/Spaced_repetition]. This is the anti-fatigue mechanism: a dormant seed appears in the weekly clustering context every week (cheap — it's just prompt content) but only demands a **numbered decision** when past its `review_on`. After ~3 re-parks (~4–5 months dormant), escalate to a forced keep-or-kill question — the backlog-research consensus is that items surviving ~12 months without sponsorship are zombies that should be retired with a recorded rationale [CITED: https://agileproductmastery.com/product-backlog-cleanup-4-steps; https://age-of-product.com/28-product-backlog-anti-patterns/].

### Pattern 4 — Persist-first, then one numbered DM

Direct reuse of the triage project-fate rule: "Proposals are durable; a scrolled-away DM never holds one alone" [VERIFIED: triage.md step 4]. The review report is a wiki page first, committed and pushed; the DM links it and carries only the numbered decisions.

### Pattern 5 — Rendered seedbed page (fixes today's live gap)

Eli literally asked where a seed was parked and nothing was visible. The wiki already has the answer-shape: ROADMAP.md is "Rendered from momo_work — do not hand-edit; the DB is the source of truth" [VERIFIED: ROADMAP.md header]. `wiki/seeds/SEEDBED.md` follows it exactly: every live seed with summary, review date, times reviewed, and provenance (atom id → item date/excerpt), plus a Retired section (status, reason, review link). Re-rendered by the triage seed procedure and by every review run. Link it from Home's Map so it's discoverable in Obsidian.

### Anti-patterns to avoid
- **Hidden/graveyard sub-backlogs** — a separate "parking" label/list that never gets processed; practitioner consensus is these make life harder, not easier [CITED: https://age-of-product.com/28-product-backlog-anti-patterns/]. The seedbed page is a *review surface with scheduled resurfacing*, not a dumping ground — the escalation rule is what keeps it from becoming one.
- **Per-seed review walk** — explicitly banned by SEED-02; also the failure mode GTD reviewers hit (item-by-item guilt tour instead of a synthesis) [CITED: https://super-productivity.com/blog/gtd-weekly-review-guide/].
- **Collector's fallacy** — capture feels productive; only processing compounds. "We may expand our knowledge permanently only by storing notes permanently" — and by *merging* collected material into decisions, not filing it [VERIFIED: fetched https://zettelkasten.de/posts/collectors-fallacy/ this session].

---

## Don't Hand-Roll

| Problem | Don't build | Use instead | Why |
|---|---|---|---|
| Semantic clustering of <100 short texts | Embeddings pipeline (pgvector, sentence-transformers, k-means/GMM) | One-pass LLM-reasoning clustering (Pattern 2) | Embedding pipelines earn their keep at scale; per-item LLM cost is the objection at thousands of documents [CITED: https://arxiv.org/pdf/2412.12459], irrelevant at tens. Embeddings also give no reasoning trace — and this project's invariant is *reasoning logged for everything*. An embeddings route would add an install (guard-gated, constraint-violating) for a worse fit. |
| Review scheduling | A scheduler daemon or custom interval engine | `review_on DATE` column + expanding-interval policy in the runbook; due-ness = one SQL predicate (`status='live' AND review_on <= current_date`) | Phase 4's tick already owns *when code runs*; this phase only needs due-ness to be a cheap DB fact. Building any trigger/timer here duplicates phase 4. |
| Lifecycle audit history | Shadow audit tables / event sourcing | Status + reason columns, no-DELETE trigger, review-report wiki pages as narrative | Four states, tens of rows, single writer. The standard audit-trigger pattern exists [CITED: https://wiki.postgresql.org/wiki/Audit_trigger] but is machinery without a customer here — the receipts requirement is satisfied by reasons-in-row + dated report pages. |
| Report delivery/formatting | Any templating/report library | Markdown by hand in the runbook, numbered-options DM convention | Conventions already exist and are HARD RULEs (plain numbered text, reply tool). |
| Date math | Custom interval logic in prompts | Postgres date arithmetic (`current_date + INTERVAL`) | The agent should output *which interval tier*, the runbook computes the date — LLMs are unreliable at calendar arithmetic; the DB is not. |

**Key insight:** this phase's entire value is judgment quality, and every piece of infrastructure that isn't a table, a prompt, or a runbook line dilutes that. The substrate already contains a proven analog for every mechanism needed (triage agent → synthesizer; goal-drill research bound → pre-work bound; proposals/PLAN-CHANGES lanes → start/merge execution; ROADMAP rendering → seedbed rendering).

---

## Runtime State Inventory

This phase migrates the seed store from string-encoded atoms to a table — the five categories, answered explicitly:

1. **Stored data keyed on the old format:** FOUND — `atoms` rows with `fate='seed'`, `fate_ref='review:YYYY-MM-DD'` → **data migration**: backfill `INSERT INTO seeds … SELECT` parsing the date from `fate_ref`, then `UPDATE atoms SET fate_ref='seed:<id>'` for each (atoms are mutable; only inbox_items are immutable — and the schema comment on `fate_ref` already anticipates "seed id" [VERIFIED: 010-ingestion.sql line 53]). Exact row count unverifiable this session (subagent Bash blocked) — planner must run `SELECT count(*) FROM atoms WHERE fate='seed';` and eyeball the fate_ref formats before writing the backfill.
2. **Live service config not in git:** FOUND — the triage runbook's step-4 seed procedure writes the old `review:` format → **code edit**: update `worksystem/triage.md` seed procedure to INSERT into seeds and set `fate_ref='seed:<id>'` in the same plan that ships the table (never leave the two formats coexisting for new writes).
3. **OS-registered state:** None — no launchd/cron exists yet for reviews (phase 4 owns it); verified by design (LOOP-01 is phase 4, ROADMAP shows phase 4 not started).
4. **Secret/env-var names:** None — verified by reading all touched runbooks; nothing in this phase consumes secrets.
5. **Build artifacts:** None — SQL + markdown only; the dashboard (phase 5) reads views and is out of scope, though the planner should note `inbox_view`/dashboard queries never referenced `fate_ref` parsing [VERIFIED: 010-ingestion.sql view definition], so nothing downstream breaks.

---

## Common Pitfalls

1. **Review fatigue → ignored reports.** Long re-listing reports stop being read; executive-report guidance converges on 1–2 pages, decisions up front, specific asks ("decision requested on X") instead of awareness language [CITED: https://projectmanagercopilot.eu/project-status-report-for-executives/]. *Avoid:* the DM carries ONLY numbered decisions + one-line arguments; everything else lives in the linked wiki report. Cap the numbered list (recommend ≤7 decisions; overflow rides to next week, most-overdue first). *Warning sign:* Eli stops answering the weekly DM.
2. **Synthesis that re-lists instead of clustering.** The lazy failure mode: the agent returns one "cluster" per seed. *Avoid:* the synthesizer's JSON contract requires cluster-level reasoning ("why these belong together, why now") and a full-coverage check (every live seed id appears exactly once across clusters/singletons — same contract-coverage rule as triage [VERIFIED: triage.md step 3]); a re-listing output fails review and re-runs. *Warning sign:* clusters of size 1 dominating the report week after week.
3. **Promotion without pre-work.** A cluster reaching Eli as a bare "these three seeds look related" pushes the thinking onto him — the system's whole point is pre-worked decisions. *Avoid:* "critical mass" is *defined* as "gets pre-work before the report": no cluster may appear as a `start` option without research + fit-vs-existing-systems + mini-brief attached. *Warning sign:* start-options in the DM with no linked brief.
4. **The graveyard drift.** Someday/maybe lists become "where dreams go to die" without prune discipline [CITED: https://super-productivity.com/blog/gtd-weekly-review-guide/]; backlogs bloat from 50 to 300+ items [CITED: https://resources.scrumalliance.org/Article/scrum-anti-patterns-large-product-backlog]. *Avoid:* expanding intervals + the 3-re-park escalation to forced keep-or-kill; recommend-kill is a first-class synthesizer output with a written reason (dismissal-with-receipts, matching triage's stance). *Warning sign:* live-seed count growing monotonically across reviews.
5. **Unbounded pre-work fan-out.** Research agents per cluster per week compounds. *Avoid:* import the goal-drill HARD BOUND — at most 2 research agents per weekly review total [VERIFIED: precedent in triage.md drill flow §4]. *Warning sign:* review-run telemetry token counts climbing week over week.
6. **Silent retirement via UPDATE.** SEED-03 dies quietly if a status can change without a reason. *Avoid:* the CHECK constraint (Pattern 1) makes it impossible at the DB layer, not just the runbook layer. *Warning sign:* none needed — it's structurally prevented; the plan-checker should verify the constraint exists.
7. **Two seed formats coexisting.** New seeds written as `review:` strings after the table ships → the due-check misses them. *Avoid:* same-plan migration + runbook edit (Runtime State Inventory §2); the review runbook's gather step can assert `SELECT count(*) FROM atoms WHERE fate='seed' AND fate_ref LIKE 'review:%'` = 0 as a drift alarm.
8. **Calendar arithmetic in the LLM.** Agents mis-add dates. *Avoid:* the synthesizer outputs an interval *tier*; psql computes the date (Don't Hand-Roll).

---

## Code Examples

All examples authored this session, following 010-ingestion.sql conventions [VERIFIED: patterns read from worksystem/010-ingestion.sql].

**Seeds table sketch (for `worksystem/011-seeds.sql`):**
```sql
CREATE TABLE IF NOT EXISTS seeds (
    id             SERIAL PRIMARY KEY,
    atom_id        INT NOT NULL REFERENCES atoms(id),      -- provenance: seed → atom → immutable item
    summary        TEXT NOT NULL,
    status         TEXT NOT NULL DEFAULT 'live'
                     CHECK (status IN ('live','merged','promoted','killed')),
    status_reason  TEXT,
    status_ref     TEXT,          -- surviving seed id / proposal path / PLAN-CHANGES path
    review_on      DATE NOT NULL,                          -- SEED-01 by construction
    times_reviewed INT  NOT NULL DEFAULT 0,
    created_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
    retired_at     TIMESTAMPTZ,
    CONSTRAINT retirement_has_reason
      CHECK (status = 'live' OR status_reason IS NOT NULL)  -- SEED-03 by construction
);
-- + no-DELETE trigger, copied from inbox_items_guard_delete's shape
```

**Backfill from the phase-2 encoding:**
```sql
INSERT INTO seeds (atom_id, summary, review_on, created_at)
SELECT id, content, substring(fate_ref FROM 'review:(.*)')::date, decided_at
FROM atoms WHERE fate='seed' AND fate_ref LIKE 'review:%';
-- then per row: UPDATE atoms SET fate_ref = 'seed:' || <seed_id> WHERE id = <atom_id>;
```

**Due-check (the fact phase 4's shell pre-check will consume; also the manual trigger):**
```sql
SELECT count(*) FROM seeds WHERE status='live' AND review_on <= current_date;
```

**Gather for the synthesis pass (one query, never per-seed):**
```sql
SELECT s.id, s.summary, s.review_on <= current_date AS due, s.times_reviewed,
       a.item_id, i.ts::date AS captured, left(i.content,120) AS source_context
FROM seeds s JOIN atoms a ON a.id=s.atom_id JOIN inbox_items i ON i.id=a.item_id
WHERE s.status='live' ORDER BY s.review_on;
-- recent knowledge context: atoms WHERE fate='wiki' AND decided_at > (SELECT max(ran_at) FROM seed_reviews)
```

**Synthesizer output contract sketch (triage-style, full coverage required):**
```json
{"clusters": [{"seed_ids": [3,7,9], "theme": "...", "critical_mass": true,
   "reasoning": "<why together, why now — argued from goals + recent knowledge>",
   "recommendation": "start|merge_park|kill",
   "change_set_target": "<existing project slug, or null>"}],
 "singletons": [{"seed_id": 4, "action": "repark", "interval_tier": 2, "reasoning": "..."},
                {"seed_id": 6, "action": "recommend_kill", "reasoning": "<the written reason>"}],
 "questions": ["1 — ..."]}
```

---

## State of the Art

| Area | Old approach | Current approach | When/why it changed |
|---|---|---|---|
| Idea parking | Flat someday/maybe lists reviewed by willpower | Scheduled resurfacing with expanding intervals + explicit prune-with-reason discipline | GTD/backlog practice converged on "unreviewed lists decay into guilt graveyards"; scheduling + pruning is the fix [CITED: GTD weekly-review guides; backlog-cleanse guidance] |
| Small-corpus text clustering | LDA / doc2vec (needs large corpora, weak on short text) | LLM embeddings + classic clustering for scale; **direct LLM reasoning for small corpora** — more distinctive and human-interpretable clusters [CITED: https://royalsocietypublishing.org/doi/10.1098/rsos.241692; https://arxiv.org/pdf/2403.15112] | LLMs made semantic grouping of short texts reliable; below ~hundreds of items the one-prompt pass is simplest and fully explainable |
| Review reporting | Activity re-lists | Decision-shaped: numbered specific asks, outcomes not busyness, 1–2 pages max [CITED: https://projectmanagercopilot.eu/project-status-report-for-executives/] | Long status reports demonstrably stop being read |
| Deprecated in-substrate | `fate_ref='review:YYYY-MM-DD'` string encoding | `seeds` table; `fate_ref='seed:<id>'` | This phase — the encoding was explicitly a phase-2 stopgap [VERIFIED: triage.md] |

---

## Assumptions Log

| # | Assumption | Section | Risk if wrong |
|---|---|---|---|
| 1 | [ASSUMED] Live seed count is small (single/low-double digits) — could not run psql (subagent Bash blocked this session) | Summary, Don't Hand-Roll | If somehow >100s of seeds exist, the one-prompt pass needs batching; trivially checked by the planner with one SELECT |
| 2 | [ASSUMED] All existing seed rows use exactly `review:YYYY-MM-DD` (runbook-specified, but actual rows unverified) | Runtime State Inventory | Backfill regex misses rows → seeds invisible to the due-check; planner must eyeball actual fate_ref values first |
| 3 | [ASSUMED] 7→14→30→60→90-day intervals + 3-re-park escalation are the right constants for Eli's tempo | Pattern 3 | Wrong constants = nagging or rotting; they're runbook text, cheap to tune — surface them in the first report so Eli can adjust |
| 4 | [ASSUMED] Weekly cadence triggered manually until phase 4 wires the tick | Open Questions | If Eli expects automation now, expectation gap — state it in the phase's completion DM |

---

## Open Questions

1. **Actual seed-row inventory.** Known: the encoding convention and at least one live seed exist (Eli asked where one was parked). Unclear: exact count and fate_ref consistency. *Recommendation:* first task of the first plan runs the SELECTs; not a planning blocker — the design holds for any small count.
2. **Trigger before phase 4.** Known: LOOP-01 owns scheduling; this phase must not build timers. *Recommendation:* ship the review as an on-demand runbook ("run the seeds review") + the due-check query as the future tick's hook; state in the report DM that weekly automation lands with phase 4. Resolved — plan to this.
3. **Seed-count ceiling.** Backlog research favors hard caps; a cap needs Eli's buy-in. *Recommendation:* no hard cap in v1 — the escalation rule bounds rot organically; mention the live count in every report so growth is visible. Resolved unless Eli objects.

No unresolved blockers for the plan-checker.

---

## Environment Availability

Direct probes (psql, command -v) were blocked for this subagent (web + read tools only), so availability is verified via substrate evidence rather than live probes:

| Dependency | Status | Evidence |
|---|---|---|
| Postgres `momo_work` with inbox_items/atoms | Available | Phases 1–2 stage=verified in ROADMAP; 010-ingestion.sql applied per triage runbook's live queries |
| Agent tool (toolless subagents) | Available | Triage + goal-driller run this pattern (phase 2 verified) |
| Discord reply tool | Available | MCP server loaded this session |
| Wiki + git push (guard-auto-approved) | Available | Home.md states wiki pushes auto-approved; goals pages exist |
| **Missing-blocking** | None | — |

Planner note: the first plan should open with the trivial live probes this research couldn't run (`\d atoms`, seed counts, fate_ref formats).

---

## Sources

**Primary (verified this session):**
- /Users/momo/momo/worksystem/010-ingestion.sql · triage.md · ingestion.md · agents/triage.md · agents/goal-driller.md — the substrate being extended
- /Users/momo/momo/wiki/projects/ingestion-loop/{PROJECT,REQUIREMENTS,ROADMAP}.md · wiki/projects/ingestion-loop-design.md · wiki/ops/work-engine.md · wiki/goals/momo-engine.md · wiki/Home.md
- https://zettelkasten.de/posts/collectors-fallacy/ (fetched + read)

**Secondary (cited, official/authoritative pages via search):**
- https://royalsocietypublishing.org/doi/10.1098/rsos.241692 · https://arxiv.org/pdf/2403.15112 · https://arxiv.org/html/2511.14528v1 · https://arxiv.org/pdf/2412.12459 (LLM clustering / thematic analysis)
- https://wiki.postgresql.org/wiki/Audit_trigger · https://supabase.com/blog/postgres-audit (audit patterns — considered, rejected as overkill)
- https://en.wikipedia.org/wiki/Spaced_repetition (interval scheduling)

**Tertiary (practitioner guidance):**
- https://resources.scrumalliance.org/Article/scrum-anti-patterns-large-product-backlog · https://age-of-product.com/28-product-backlog-anti-patterns/ · https://agileproductmastery.com/product-backlog-cleanup-4-steps (backlog rot)
- https://super-productivity.com/blog/gtd-weekly-review-guide/ · https://get-alfred.ai/blog/gtd-weekly-review (GTD review discipline)
- https://projectmanagercopilot.eu/project-status-report-for-executives/ (decision-shaped reports)

## Metadata

| Area | Confidence | Reason |
|---|---|---|
| Substrate integration (schema shape, runbook seams, conventions) | HIGH | Every touched file read this session; extension points explicit in the code |
| Clustering approach | HIGH | Constraint-driven: small corpus + reasoning-logged invariant + no-install rule all independently point at the same answer; literature agrees |
| Lifecycle schema | HIGH | Directly mirrors proven phase-1 by-construction patterns |
| Review scheduling constants | MEDIUM | Sound pattern (spaced repetition + backlog pruning), constants are judgment calls flagged for Eli |
| Report design | MEDIUM | Qualitative practitioner guidance; strong convergence but no hard data |
| Live DB state | LOW | Could not probe (tool policy); flagged as the first plan's opening task |

**Researched:** 2026-07-03 · **Valid until:** ~2026-08-02 (stable internal domain; the only fast-moving input is the DB's live state, which the planner re-probes anyway)
