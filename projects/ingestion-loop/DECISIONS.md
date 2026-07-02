# Decision Log — Ingestion + Autonomous Loop

> Autonomous-mode log (same regime as the dashboard's DL series; this project uses IL-NN).
> Format: Prompt → Context → Decision → Implications → Priority/Reversibility.

## IL-01 · Phase-1 substrate designed orchestrator-side (DL-16 pattern)

- **Prompt:** Inbox/atom schema is engine architecture — planner implements, orchestrator designs.
- **Decision:** `inbox_items` (typed, append-only) + `atoms` (kind, fate, reasoning, provenance FK) + `capture_item()`/`materialize_atoms()` + `inbox_view`. **Immutability is a DB trigger** (UPDATE of content-bearing columns and ALL DELETEs raise) — "never destroyed" by construction, not discipline. Extraction handlers normalise every input type to text upstream; ONE extractor agent works on text — "per-type handler, one atom model" with the smallest possible surface.
- **Implications:** Re-ingestion = a new extraction pass over the same immutable item; old atoms remain (they carry their own fates); provenance never breaks. Fates stay `pending` in phase 1 — triage (phase 2) owns fate decisions, keeping the phases honestly separable.
- **Priority/Reversibility:** High (foundation); additive schema, reversible.

---
*Started 2026-07-03.*
