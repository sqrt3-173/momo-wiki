# Phase 4 run record — go-live receipts + overnight proof

Written strictly from receipts (DB rows, logs, message ids) — never session prose.
Go-live: 2026-07-03 13:51:14 (engine_state → active; go-live ask = Discord msg 1522449575519195257).

## Proven before the overnight window (receipts inline)

- **LOOP-01 — near-free scheduled idle ticks: PROVEN.** Stamp-driven ticks with no human and no
  force file at 14:08:01 and 14:39:00 (1,859s apart ≈ TICK_INTERVAL + cron granularity), both
  `idle — no work`, both one psql round trip; `agent_runs` count unchanged (0 rows > id 13)
  through both. [tick.log trigger-type lines; SELECT count(*) FROM agent_runs WHERE id > 13 → 0]
- **LOOP-02 — governed one-unit execution: PROVEN (manual-trigger E2E, run 13).** Forced tick
  13:38:01 → fresh non-bare session → claimed the triage unit → 21/21 real atoms decided
  (10 dismissed / 9 wiki / 2 project proposals), 1,225,112 tokens, cost_json 4.91, outcome ok;
  fingerprint BLOCK line in guard.log at 13:38:23 (the session's own guard firing); bridge PIDs
  identical before/during/after (74228/74262); 3 notifications queued, zero Discord attempts;
  drained by the interactive session and fetch-verified (msg 1522449370438435006).
  **Honest finding en route:** run 12 (13:31) was killed by its own `--max-budget-usd 2` at
  $2.23 mid-agent with zero output landed — bounds work, the constant was wrong; raised to $5
  and documented in heartbeat.md's constants. Quiesce during run 13's window was imperfect
  (interactive wiki commits ran mid-window); the guard evidence therefore rests on the
  fingerprint BLOCK line, which only the tick session can produce.
- **LOOP-03 — pause/resume from Eli's phone: PROVEN (composite).** Eli typed "pause"
  (msg 1522455220347600987) → engine_state paused at 14:13:48; "resume"
  (msg 1522455784242417664) → active at 14:15:54. No scheduled tick landed inside his 2-minute
  window, so the paused-skip-during-his-window wasn't observed live; the skip behavior itself
  is on record from the forced paused tick at 13:28:01 (`paused — skip`, no session, receipts
  updated). Flag mechanics + skip behavior each proven; stated honestly as composite.
- **LOOP-04 — wrapper-owned telemetry: PROVEN.** Runs 12 and 13 both have complete rows
  (opened before launch, closed after, tokens digit-for-digit from the JSON usage, cost_usd
  NULL by the unpriced-Fable-5 rule, notes carrying exit code + cost_json) — including run 12,
  which DIED mid-session and still has a full record. Breadcrumbs (`agent_events`) present for
  both.

## The overnight window (success criterion 5) — PENDING

Awaiting Eli's bedtime dump. Protocol: capture + ✓ ack per ingestion.md, then nobody converses
until morning; scheduled ticks own all work. This section gets the full trace tomorrow:
item id/ts → atoms → fates with reasoning → wiki commits → every overnight agent_runs row →
notifications queued → tick.log window (idle passes included) → per-criterion verdict.

## Success-criteria scoreboard (as of go-live day)

| # | Criterion | Status |
|---|---|---|
| 1 | Idle tick = shell pre-check, near-free, no session | **LIVE-proven** (2 scheduled idle ticks) |
| 2 | Work → fresh session claims exactly one unit, full discipline, clean close | **LIVE-proven** (run 13; manual trigger) |
| 3 | pause/resume from Eli respected by the pre-check | **Proven composite** (flag from his typed words + skip on record) |
| 4 | Every tick session in telemetry + outcomes in the digest | **LIVE-proven** (runs 12+13; digest tomorrow completes it) |
| 5 | Overnight proof: dump before bed → filed by morning, nobody in conversation | **PENDING tonight** |
