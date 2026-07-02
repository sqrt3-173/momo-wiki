# Decision Log — Ingestion + Autonomous Loop

> Autonomous-mode log (same regime as the dashboard's DL series; this project uses IL-NN).
> Format: Prompt → Context → Decision → Implications → Priority/Reversibility.

## IL-01 · Phase-1 substrate designed orchestrator-side (DL-16 pattern)

- **Prompt:** Inbox/atom schema is engine architecture — planner implements, orchestrator designs.
- **Decision:** `inbox_items` (typed, append-only) + `atoms` (kind, fate, reasoning, provenance FK) + `capture_item()`/`materialize_atoms()` + `inbox_view`. **Immutability is a DB trigger** (UPDATE of content-bearing columns and ALL DELETEs raise) — "never destroyed" by construction, not discipline. Extraction handlers normalise every input type to text upstream; ONE extractor agent works on text — "per-type handler, one atom model" with the smallest possible surface.
- **Implications:** Re-ingestion = a new extraction pass over the same immutable item; old atoms remain (they carry their own fates); provenance never breaks. Fates stay `pending` in phase 1 — triage (phase 2) owns fate decisions, keeping the phases honestly separable.
- **Priority/Reversibility:** High (foundation); additive schema, reversible.

## IL-02 · First checker "revise": four fixes applied orchestrator-side, not via planner re-run

- **Prompt:** Phase-1 checker returned `revise` — blocker: `file` item type had no extraction handler (4-of-5 type coverage on ING-04); warnings: immutability truths overclaimed ("no one can" — owner can disable triggers), extractor run mechanism unpinned (bash `claude -p` would hit a non-bypassable guard block), re-ingestion claimed but never asserted.
- **Decision:** All four fixes were line-level with prescriptive fix_hints, so applied directly to the plan JSON (file handler added to both artifacts; truths reworded honest — trigger holds under normal DML, the guard's ALTER ask is the compensating control; extractor pinned to the Agent tool as toolless prompt-only subagent; re-ingestion acceptance step added: second pass appends, originals byte-identical, status re-mark passes the trigger). GSD's loop sends issues back to the planner — for one-liner prescriptive fixes an orchestrator patch + this log entry is the pragmatic equivalent; structural issues would go back to the planner properly.
- **Priority/Reversibility:** Process precedent worth keeping; low risk.

## IL-03 · Phase-2 checker revise: 2 blockers + 5 warnings, all applied (IL-02 mechanism)

- **Blockers:** (1) plan 2 silently depended on plan 1's goals pages while sharing its wave — re-waved to a strict 1→2→3→4 chain; (2) project-idea fates were ephemeral (a DM + a working_title fate_ref resolving to nothing) — now persisted FIRST as durable proposal pages (`wiki/projects/proposals/`), fate_ref = the path, DM links it. A scrolled-away message never holds a pending proposal.
- **Warnings applied:** seed fate_refs machine-parseable (`review:YYYY-MM-DD`) so phase 3 can query them; goals-page synthesis carries per-section `source:` comments (nothing-invented is a checkable trace); question-held atoms get a `HELD <date>:` reasoning prefix (queryable awaiting-Eli state) + fetch_messages send-verification; an "on answer" resolution step (held atoms always have an owner); plan 4 reworded to the run-time pending set — reality-check confirmed drift (22 → 26 pending while planning).
- **Priority/Reversibility:** the durable-proposals fix is load-bearing for the receipts principle; the rest hygiene. All reversible.

---
*Started 2026-07-03.*
