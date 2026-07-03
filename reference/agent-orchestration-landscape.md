# Agent orchestration landscape (2026-07-03)

<!-- Filed by triage run 13 from Eli's adversarially-verified landscape report
     (inbox item 10, file: inbox-files/2026-07-03-eli-agent-orchestration-landscape.md).
     Each section carries atom provenance. Companion cost anchors live in
     [[../ops/model-cost-reference]]; the two project proposals this report spawned are
     Director's Console and Proactive Ideation Loop (wiki/projects/proposals/). -->

## Engine choice
<!-- atom 42 · item 10 · 2026-07-03 -->
Landscape report 2026-07-03 (adversarially verified): Claude Code + Claude Agent SDK is
the recommended agent engine. Lifecycle hooks (PreToolUse with allow/deny/ask/defer) are a
supported, first-class approval-gate primitive — this validates MOMO's existing guard
architecture (ops/momo-guard.py, see [[../ops/guardrails]]) as the pattern to build
console/fleet approval gates on.

## Always-on fleet runtime options
<!-- atom 43 · item 10 · 2026-07-03 -->
Verified options for an always-on fleet runtime: (1) Trinity — open-source,
Docker-isolated, cron + persistent SQLite backlog, 74 MCP tools so an interactive session
can direct the fleet; (2) Anthropic Managed Agents — hosted, public beta since Apr 2026.
Agent Teams is experimental and not always-on. Relevance: these are the candidate
substrates if the session-bound engine (current binding-constraint hypothesis, see
[[../goals/momo-engine-strategy-2026-07-03]]) proves too slow once runs are priced.

## Multi-model routing
<!-- atom 44 · item 10 · 2026-07-03 -->
LiteLLM is the verified choice for multi-model routing and failover. Only needed if
per-chat model selection in the Director's Console outgrows a direct model picker; not a
present dependency.

## Doctrine: fan out reads, serialize writes
<!-- atom 45 · item 10 · 2026-07-03 -->
Strongest-evidence doctrine from the report: "fan out reads, serialize writes."
Multi-agent beat single-agent by 90.2% on research evals, but parallel-writer coding fails
via decision fragmentation. Production shape: map-reduce-and-manage — parallel
research/read fan-out, a single serialized write stream. Direct bearing on MOMO's standing
parallel-executor guard decision (Eli's call): serial execution (DL-07) is the
evidence-backed default for writes; parallelism should be spent on the read side.

## Verification design
<!-- atom 46 · item 10 · 2026-07-03 -->
MAST study: 41-86.7% task failure across 2025 open-source multi-agent frameworks, with
verification the structural weak point. Design implication: multi-level verification,
including fresh-context reviewer agents that share NO context with the writer. This is the
evidence base for MOMO's existing council-review/checker pattern; enforce the
no-shared-context property when spawning reviewers.

## The honest ceiling
<!-- atom 48 · item 10 · 2026-07-03 -->
Honest read of the "100 FTE" framing: agent fleets give 100-person breadth on READS but
only single-digit parallel WRITE streams, and the real throughput ceiling is the
director's own review capacity. Implication: the highest-leverage build is whatever raises
Eli's review throughput (briefing view, approval inbox, risk-ranked queue — see the
Director's Console proposal), not more parallel writers.

## Open questions (unverified as of 2026-07-03)
<!-- atom 49 · item 10 · 2026-07-03 -->
The report explicitly could NOT verify: (1) framework wrap-ability; (2) concrete monthly
economics — Max subscription vs API vs Managed Agents (bears on Eli's pending engine
budget-ceiling question); (3) evidence for cross-model adversarial review; (4) whether any
product ships proactive ideation with approval gates. Treat these as open research
questions, not settled facts.
