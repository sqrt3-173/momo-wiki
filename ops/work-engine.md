# The Work Engine (the task/project system)

The **Work** layer of the three-part architecture ([[work-engine|Work]] · Knowledge = the wiki ·
Conversation = Discord). It's the keystone that turns MOMO from "responds when asked" into "always
advancing every project." Project-agnostic; holds ALL projects. **Built + proven 2026-07-01.**

## Hierarchy
`project → module → task → (subtask)` — flexes to project size:
- **Small project:** flat — tasks with `module_id = NULL`.
- **Big project (e.g. NV Health CRM):** decomposes into **modules** (functional areas: Twilio, Calendar
  bookings, Lead flow…), each holding tasks; tasks can nest via `parent_task_id` for subtasks.

## Store
- Its own Postgres database **`momo_work`** (separate from any project's data, incl. the CRM).
- Schema/functions live in `/Users/momo/momo/worksystem/`: `schema.sql`, `functions.sql`, `test-seed.sql`.
- Tables: **`projects`** (slug, name, status, priority, discord_channel) → **`modules`** (project_id, name,
  status, priority, position) → **`tasks`** (project_id[denorm], module_id, parent_task_id, title, status
  [todo/doing/blocked/done/cancelled], priority 1–5, `depends_on` int[], `scheduled_for`, `model_tier`,
  `claimed_by`, `claimed_at`, `result`).

## The engine's core mechanics (all tested + working)
- **Atomic claiming** — `SELECT * FROM claim_next_task('<host>')` atomically claims the next *ready* task
  (todo · schedule due · all deps done · highest priority) using `FOR UPDATE SKIP LOCKED`. **Two hosts
  (mini + macbook) call it simultaneously and never grab the same task** — the DB guarantees it. This is
  what makes the 2-host overnight setup safe.
- **Dependencies** — a task with `depends_on` isn't claimable until every listed task is `done`
  (proven: a webhook task stayed blocked until its Twilio-account dep completed, then auto-unblocked).
- **Module parallelism** — independent modules' tasks are claimable at once (proven: mini + macbook
  claimed Twilio and Calendar tasks in parallel). Orchestration fans agents across independent work.
- **Model tiering** — each task carries `model_tier` (frontier/workhorse/cheap/bulk/local) → route to the
  right-cost model per [[model-cost-reference]].
- **Rollup views** — `project_status` and `module_status` give done/doing/todo/blocked counts per
  project + module (for status + Discord reports).
- **Human-in-the-loop** — each task carries `assignee` + `assignee_type`. Agent tasks (momo/nunu) auto-claim;
  **human tasks (eli/yana) are excluded from auto-claim and surfaced instead** via `human_queue(person)`
  (their pending tasks, ranked by how much each is *blocking*). Dependencies cross assignees: an agent task
  can `depend_on` a human task → stays blocked (MOMO flags what it's holding up), and auto-unblocks the moment
  the human marks theirs done. So a human never silently stalls a project; the machine waits correctly, then
  flows. Marked done the natural way — the person tells MOMO in Discord. **Proven end-to-end 2026-07-01.**

## Built vs. next
**✅ Built + proven:** the store (3 tables, indexes), `claim_next_task()`, rollup views, dependency +
priority + atomic-claim logic — all validated end-to-end. Seeded with NV Health as a structural example.

**⬜ Next (the loop + surrounds):**
1. **The work loop** — MOMO (each host, always-on): `claim_next_task` → execute (fanning out sub-agents,
   using the right model tier) → mark done + write `result` → report to the project's Discord channel → repeat.
2. **A thin CLI/helper** so MOMO adds projects/modules/tasks + reports cleanly (vs raw SQL).
3. **Inbox + triage** — capture point (Discord `#inbox` / wiki inbox) → MOMO triages into Knowledge (wiki) or Work (tasks).
4. **Per-project Discord channels** — each active project reports in its own channel.
5. **Cost tracking + agent telemetry** — per-task spend + agent-run events (feeds any future dashboard/visual).

## Telemetry — MANDATORY orchestrator lifecycle practice (2026-07-02, USE-01/DL-16)

Every run gets recorded — main-loop work and subagents alike. Not optional; this is how the
dashboard's agents/usage/digest views stay true. Writer trio on `momo_work` (see
`worksystem/008-telemetry.sql`; momo writes, `dashboard_ro` only reads the views):

- **Spawn/claim** → `SELECT start_run('<host>','<kind>',<project_id>,<plan_id>,<task_id>,'<tier>');` → keep the id.
- **Progress** → `SELECT log_event(<id>,'step','<what just happened>');` — every call bumps the
  heartbeat; no heartbeat for 3 min ⇒ the dashboard honestly shows the run as **stale**.
- **Commits & DMs** → also breadcrumb them with dedicated kinds so the digest can surface them:
  `log_event(<id>,'commit','<repo>: <subject>')` and `log_event(<id>,'dm','<one-line summary>')`.
- **Completion** → `SELECT finish_run(<id>,'ok'|'error'|'killed',<tokens>,<cost_usd>,'<note>');`
  - `tokens` = the harness notification's total, digit-for-digit. Not reported ⇒ NULL.
  - `cost_usd` = tokens × the model's **blended rate** from [[model-cost-reference]] (e.g. Sonnet 5
    ≈ $4/M ⇒ 50,000 tok = 0.2000), computed at write time — rates NEVER live in SQL. Model not in
    the reference (currently: Fable 5) ⇒ cost NULL + note; **missing renders absent, never 0**.

## Guard note
Schema/DB creation used `#MOMO_OK` on the psql commands (Eli's explicit "GO" for the build). Ongoing task
mutations by MOMO will hit the same psql-mutation guard — decide whether to allowlist `momo_work` writes
(like the wiki-push exemption) once the loop is live, so MOMO can update task status unattended.
