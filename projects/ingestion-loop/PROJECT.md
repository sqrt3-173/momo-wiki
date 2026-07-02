# Ingestion + Autonomous Loop

> GSD-style living project context. Source spec: [[../ingestion-loop-design]] (the Discord
> design session, 2026-07-03 — binding). Requirements: [[REQUIREMENTS]] · Roadmap: [[ROADMAP]].

## What This Is

The always-on layer of the work engine. Eli dumps anything (notes, transcripts, images,
videos-to-ingest); the machine decomposes dumps into atoms, reasons about each against his
businesses' goals, files knowledge, creates/advances projects and tasks, parks and re-reviews
seeds — and a scheduled heartbeat keeps the engine working when no one is in a conversation.

## Core Value

Eli dumps freely and sleeps; the machine turns the pile into filed knowledge, advanced
projects, and decisions worth making — with receipts for everything.

## Requirements

See [[REQUIREMENTS]] — v1 across ING / TRI / GOAL / SEED / LOOP / VIS.

### Out of Scope (v1)

- Parallel executor fan-out (separate guard decision Eli must make).
- Auto-approval of hard-gated actions (spend/publish/install/delete/contact) — never.
- Automatic speech-to-text (voice memos arrive as transcripts for now; STT is its own seed).

## Constraints

- Substrate: the existing engine (momo_work, wiki, Discord, guard, telemetry, dashboard).
- Originals immutable — inbox rows are never edited or deleted (append-only record).
- Triage is REASONING against the goals layer — no pattern-rules engine (Eli's explicit call).
- Loop ticks must be near-free when idle: shell pre-check before any Claude session wakes.
- The guard governs tick sessions exactly as it governs me (same settings, same categories).

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| D-01 Decompose-then-dispose: atoms carry fates, items never do | one dump holds many meanings (Eli killed fate-per-item) | — Pending |
| D-02 Goals-as-context triage, no rules list | rules only catch the pre-imagined; reasoning derives relevance + magnitude | — Pending |
| D-03 Immutable inbox, full provenance both directions | audit + re-ingestion when context changes | — Pending |
| D-04 Cheap-check-first heartbeat (shell → psql → only then Claude) | idle ticks must cost ~nothing | — Pending |
| D-05 Action boundary unchanged: draft/research autonomous, contact/spend/publish never | authority, not intelligence | — Pending |

---
*Created 2026-07-03 at Eli's go.*
