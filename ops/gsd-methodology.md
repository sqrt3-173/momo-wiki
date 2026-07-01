# GSD methodology — faithful extraction (for native replication in the Work engine)

Deep-ingested from `github.com/open-gsd/gsd-core` (the spec-driven AI-workflow framework — NOT the browser
tool). Source of truth for replicating GSD natively in the [[work-engine]]. Ingested 2026-07-01.

## What GSD is
A context-engineering + spec-driven framework that drives AI coding agents through a disciplined loop,
fighting "context rot" by running heavy work in **fresh-context subagents** (orchestrator stays lean ~10–15%,
delegates by passing **paths not contents**). Runtime-agnostic (Claude Code / Codex / Gemini / opencode).

## Granularity (the structure)
`Project → Milestone (vX.Y) → Phase (integer; decimal e.g. 2.1 = urgent insertion) → Plan ({phase}-{plan}) → Task`
- **Plan** = the **execution unit**: a *vertical slice*, 2–3 tasks, sized ~50% context, run by ONE fresh-context executor → produces a SUMMARY.
- **Task** = a step inside a plan, with **acceptance criteria as a hard gate** (executor runs the grep/file/CLI check; max 2 fix attempts then escalate).
- **Wave** = parallel-execution grouping: plans with deps met + non-overlapping `files_modified` run in parallel. Pre-computed in PLAN frontmatter.
- **→ Work-engine mapping:** add a **`plans`** level (phase → plan → task); a Work-engine "task" that an agent claims+executes ≈ a GSD **Plan** (vertical slice), and its subtasks ≈ GSD **tasks** (steps). Milestones + phases as designed.

## The lifecycle (the phase loop)
**new-project** (once) → per phase: **discuss → ui (frontend only) → plan → execute → verify → ship** → **complete-milestone** (per group).

- **new-project** → `config.json`, `PROJECT.md`, optional research (4 parallel researchers → synthesizer), `REQUIREMENTS.md` (REQ-IDs like `AUTH-01`), `ROADMAP.md` + `STATE.md` (via `gsd-roadmapper`).
- **discuss-phase N** → `{N}-CONTEXT.md`: scout codebase, generate 1–2 specific ambiguities/category, Q&A to lock decisions (`D-01`…), mandatory `<canonical_refs>` (paths to every spec).
- **ui-phase N** (frontend) → `{N}-UI-SPEC.md` (locks spacing/type/color/copy before planning); gate blocks planning if UI phase + no spec.
- **plan-phase N** → pipeline **researcher → planner → checker** (revision loop max 3): `RESEARCH.md` → `{N}-{nn}-PLAN.md` files → `VERIFICATION.md`. Gates: **Requirements Coverage** (every roadmap REQ-ID in ≥1 plan) + **Decision Coverage** (every CONTEXT decision referenced — blocks).
- **execute-phase N** → discover plans → group into **waves** → per plan spawn **`gsd-executor`** (worktree-isolated, background), dispatched sequentially; checkpoints pause→fresh continuation agent; between waves: merge worktrees + **build & test gate** + update STATE/ROADMAP once/wave. Then spawn **`gsd-verifier`**.
- **verify-work N** → human UAT (`{N}-UAT.md`): present one test at a time, classify pass/skip/blocked/issue (severity inferred from words); issues → parallel debug agents → planner `--gaps` → execute `--gaps-only`.
- **ship N** → preflight gates (verification exactly `passed`, clean tree, security `threats_open==0`) → PR body (per-plan changes, requirements, TDD audit) → `gh pr create`. Ship ≠ merge/tag.

## Quality gates ("done" definitions)
- **Task done** = acceptance-criteria hard gate passes.
- **Plan done** = SUMMARY.md written + committed atomically.
- **Phase done** = **goal-backward** — verifier checks `must_haves` (truths + artifacts + key_links) against the *real codebase*, not task completion ("task completion ≠ goal achievement"). Status `passed`/`gaps_found`/`human_needed`.
- **Ship** = verification `passed` + clean + security clear.

## Human-alignment gates (config.json → gates, all default true)
`confirm_project` · `confirm_phases`/`confirm_roadmap` · `confirm_breakdown` · `confirm_plan` · `execute_next_plan` · `issues_review` · `confirm_transition`. Plus safety: `always_confirm_destructive`, `always_confirm_external_services`. `--auto`/`--chain` auto-approve + chain.
- **→ Work-engine mapping:** each gate = a **human task** (assignee=eli/yana) that the next agent phase `depends_on` → blocks until signed off. Exactly the human-in-the-loop layer already built.

## Documents (all under `.planning/`) — map to the three layers
- **Knowledge (→ wiki):** `PROJECT.md` (what/why/core-value/decisions), `REQUIREMENTS.md` (v1/v2/out-of-scope + traceability table), `research/*` (STACK/FEATURES/ARCHITECTURE/PITFALLS/SUMMARY), per-phase `CONTEXT.md`, `RESEARCH.md`, `UI-SPEC.md`, `SUMMARY.md`, `VERIFICATION.md`, `UAT.md`.
- **Work (→ DB):** `ROADMAP.md` → milestones/phases/plans/tasks rows; `PLAN.md` frontmatter (`phase,plan,wave,depends_on,files_modified,autonomous,requirements,must_haves`) → task fields; `STATE.md` → **replaced by the live DB** (status/progress derive from task rows).
- **config.json** → per-project config (gates, workflow toggles, parallelization).

## Engine
`src/*.cts` (`gsd-tools.cjs` / `gsd_run query <op>`) does deterministic file mutations (state/roadmap parsing, phase locating, config, commit). **→ our equivalent = SQL/functions on `momo_work`** (the DB is deterministic by nature).

## Repo scale (be pragmatic)
99 commands, ~144 workflow files, 34 templates, 78 references, 35 subagent roles, a TS engine. **The CORE loop + docs + gates + granularity (above) is ~80% of the value and fully captured.** Much of the 99 (north-star ideation, mempalace KG, forensics, stats, secure/audit suite) is peripheral — replicate the core faithfully, add periphery on demand.

## Gaps (flagged honestly, from the ingestion)
- `templates/research.md` literal skeleton (structure known, exact tags not retrieved — fetcher refused).
- `references/gates.md` verbatim (gate list authoritatively from config.json instead).
- Full bodies of some agent defs (planner/verifier/plan-checker) + peripheral templates + the `references/` deep-dives + the `ns-*` sub-suite — inferred from names/parents, not opened. **The core is sufficient to rebuild; go deeper per-piece as we build each part.**
