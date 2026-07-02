<!-- First goal-drill strategy brief. Produced by worksystem/agents/goal-driller.md during
     the phase-2 contract test on the momo-engine Direction statement; preserved because the
     analysis is load-bearing, not a test artifact. Linked from [[momo-engine]]. -->

# Strategy — MOMO Engine: "the machine works while Eli sleeps" (2026-07-03)

## Binding constraint (hypothesis)
**The engine is session-bound.** With no heartbeat, nothing happens while Eli sleeps — the
goal's core promise fails at 100% regardless of capture/triage quality. Every other driver is
live or in flight (capture verified, triage executing, dashboard done, guard live); this one
is at zero. Hard dependency making it worse: always-on cannot responsibly switch on while
runs are unpriced (DL-17) and no budget ceiling exists — an unattended loop with unmeasured
spend violates the engine's own idle-cost constraint. **Constraint = the heartbeat, gated by
cost visibility. Not pipeline quality. Not throughput.**

## Drivers → known facts
1. **Always-on operation** — session-bound today; heartbeat is roadmap phase 4 (todo); idle
   must cost ~nothing (goals constraint).
2. **Dump-to-knowledge pipeline** — phase 1 verified, phase 2 executing, phase 3 todo.
3. **Autonomous advancement** — serial execution (DL-07); parallel fan-out awaits Eli's
   guard decision; live workloads exist (ingestion-loop, nv-health, skip-bin parked).
4. **Cost discipline** — telemetry live but all runs unpriced (Fable 5 rate missing, DL-17);
   no run-rate ceiling recorded.
5. **Trust & visibility** — dashboard v1 complete; guard governs all sessions.

## Research gaps
Fable 5 token rate · Eli's run-rate ceiling · whether scheduled headless sessions inherit
the guard identically (established for `claude -p` from the work dir; unverified for
launchd/cron invocation) · real overnight dump volume (sizes whether serial is a bottleneck).

## Proposed moves (each names what it displaces)
1. **Price the engine** — record the Fable 5 rate, backfill telemetry cost_usd, get a
   ceiling from Eli. *Takes:* rate lookup + backfill + one question. *Displaces:* dashboard
   design-council round-2 backlog.
2. **Re-sequence: heartbeat (phase 4) before seeds (phase 3)** — ship the loop as soon as
   triage verifies. *Takes:* scheduled trigger + pre-check + guard-binding verification.
   *Displaces:* seeds/weekly synthesis slip one phase (dumps already capture safely).
3. **Ship the heartbeat serial-only; defer the parallel guard decision until priced
   telemetry proves serial is too slow.** *Takes:* a recorded decision + a telemetry check
   after the first priced week. *Displaces:* parallel throughput, deliberately — and takes
   the standing-debt decision off Eli's plate until data forces it.

## Questions for Eli (sent 2026-07-03)
1 — Monthly ceiling for an always-on engine ($50 / $150 / $300 / no ceiling, weekly report)?
2 — OK to pull the heartbeat ahead of seeds so the engine runs while you sleep sooner?
3 — Defer the parallel-executor guard decision until priced telemetry shows serial is a bottleneck?
