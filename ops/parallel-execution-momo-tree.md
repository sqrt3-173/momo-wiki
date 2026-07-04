# Parallel execution — the recursive MOMO tree (architecture decision, 2026-07-04)

Worked out with Eli. The path from "one main MOMO + read-only helpers" to "N parallel builders."

## The reframe (Eli's insight — correct)
The safety boundary is **the guard** (`ops/momo-guard.py`), NOT "main-loop vs subagent". The guard
already governs subagents + `claude -p` runs from the workdir (same settings). So a **write-capable
sub-agent under the same guard is NOT "full rein"** — it has the same catastrophe protection I do
(can't spend money, change system settings, delete data outside the workdir). Giving helpers write
access ≠ removing safety; it's **replicating the governed structure one layer down**: main MOMO →
sub-MOMOs (guard-governed, write-capable) → their read-only helpers. Sound design.

I initially over-framed write-access as "full rein" — that was wrong. Corrected.

## What actually needs building (the 3 missing pieces — NOT permission handouts)
1. **Hierarchical approval routing.** The guard's ASK tier DMs Eli (push? install? delete data?). One
   agent asking occasionally = fine; 10 sub-MOMOs pinging Eli = he drowns. Fix: **sub-MOMOs ask the
   MAIN MOMO, which adjudicates**; only genuinely human-needing things reach Eli. (This IS the two-layer
   structure — made concrete.)
2. **Sandboxed worktrees + merge/review gate.** 10 MOMOs editing the same files collide. Each works on
   its own isolated copy (the Agent tool's `isolation: "worktree"` already exists), then the main MOMO
   merges. A **review gate is required** because the guard blocks *catastrophe* but ALLOWS ordinary edits
   in the workdir — so 10 agents can still make an ordinary mess inside the allowed zone; review catches it.
3. **Untrusted-data agents stay read-only.** The agents whose job is "read the web / read random files"
   are the prompt-injection surface → they STAY declawed. Only agents **building from a trusted plan**
   become write-capable sub-MOMOs. So the tree is mixed: sub-MOMOs (trusted-plan builders) + read-only
   helpers (untrusted-data readers) under each.

## Why this matters
This is the concrete shape of the "smart orchestration of agents" governing goal + the answer to Eli's
"if I had my M3 Max, would it be faster?" (real parallel building needs this layer). Sequenced AFTER the
NV deadline. Depends on / overlaps [[../goals/momo-engine-hardening-2026-07-03]] (scoped roles + egress +
verification layer are pieces 2 & 3). See also [[work-engine]], [[gsd-workengine-design]].
