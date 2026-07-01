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

## Guard note
Schema/DB creation used `#MOMO_OK` on the psql commands (Eli's explicit "GO" for the build). Ongoing task
mutations by MOMO will hit the same psql-mutation guard — decide whether to allowlist `momo_work` writes
(like the wiki-push exemption) once the loop is live, so MOMO can update task status unattended.
