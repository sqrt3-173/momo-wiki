# Proposal — Director's Console

<!-- atom 40 · item 10 · 2026-07-03 · triage run 13. Status: APPROVED by Eli 2026-07-03 (1a/2a) — project row live behind a confirm_roadmap gate; was: AT ELI'S GATE (D-05) —
     proposed, not created. Source report: inbox-files/2026-07-03-eli-agent-orchestration-landscape.md.
     Evidence base: [[../../reference/agent-orchestration-landscape]]. -->

**What:** a thin director's console — a web chat interface where Eli spins up fresh
MOMO-powered chats on a whim, unbound by Discord channel setup, with per-chat model
selection (sonnet/opus by task), plus the fleet-direction surfaces the landscape report
identifies as the genuinely novel build layer: briefing view, approval inbox, risk-ranked
review queue.

**Why:** the verified throughput ceiling of an agent fleet is the director's own review
capacity — this is the multiplier on Eli himself, and he explicitly asked for the chat
piece (atom 39, same report).

**Rough scope:** Next.js app on the existing stack + Claude Agent SDK sessions +
integration with the existing guard/approval boundary; LiteLLM (verified routing choice)
only if per-chat model selection outgrows a simple picker.

**Why now:** the landscape report just verified the engine stack (Claude Code + Agent SDK,
PreToolUse gates) and found nothing off-the-shelf for this layer; files at Eli's gate as
the leading next-project candidate once ingestion-loop ph4 (heartbeat) and ph5
(visibility) land — ph5's visibility goal and this console likely converge and should be
scoped together.

## The core screen: the decision queue (Eli's refinement, same day)
<!-- item 11 · 2026-07-03 — Eli's design input while the proposal sat at his gate -->
Eli's sharpening: decisions shouldn't arrive as chat prose at all. The console's primary
surface is a **ranked decision queue** where each open decision is a card carrying:
- the CONTEXT, pre-worked — what it's about, the receipts behind it, what each option
  displaces (pulled from gates + the notifications queue, which already carry this data);
- the OPTIONS as one-tap answers (the Claude Code question UX, but persistent — a queue
  that waits, not a prompt that scrolls away);
- **a "discuss" branch** — one tap spawns a fresh MOMO chat scoped to that single
  decision's context; the discussion can end by answering the gate, and the thread is
  archived against the decision as its reasoning record.
Ranked by blast radius (the guard's risk categories, reused): irreversible/money/public
first, roadmap-shaping second, routine batched. The substrate already exists — gates are
rows, notifications are rows, both dashboard-readable; this screen makes them answerable.
