# Feedback: model tiering — Fable orchestrates, workers execute (Eli, 2026-07-08 21:19)

**What happened:** Eli asked whether Fable 5 was reserved for high-level orchestration. Audit
said no: tick sessions launched on Fable (verified in run logs' modelUsage) and most overnight
subagents inherited Fable from the main session.

**Why:** frontier usage draws down the subscription's rate limits (the real overnight constraint
— July 3 blackout). Execution of already-checked plans is worker-tier; the intelligence lives in
the plans and gates, which Fable writes.

**How to apply:**
- Ticks: pinned in code — `momo-tick.sh` launches `--model claude-sonnet-5` (commit 313185e).
- Subagent spawns (every Agent call): set `model` EXPLICITLY — haiku for mechanical work, sonnet
  for research/drafting/execution, Fable (omit) ONLY for orchestration-grade judgment or the
  hardest adversarial verification. Never let workers inherit Fable by default.
Related: [[report-usage-not-dollars]], [[../ops/work-engine-v2]].
