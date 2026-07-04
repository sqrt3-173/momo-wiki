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

## Approval model — the Director's Console tolerance dial (Eli, 2026-07-04)
Eli's refinement: he doesn't mind BEING asked IF it's a tunable Console queue, not Discord spam. So:
- **Tolerance dial** in the Director's Console: start at 0 (show me everything), raise as trust grows,
  lower if nervous. Eli controls the volume, not a fixed rule.
- **Main-MOMO filter UNDERNEATH the dial** — batches the purely mechanical stuff so even "show everything"
  isn't literally every file save; the dial controls what gets past the filter to the Console.
- **Don't-stall fallback** — if Eli's away and an agent waits on a yes: auto-handle below a stakes line he
  sets, else the parent MOMO decides. Agents never freeze on a human. (This is the [[../goals/director-console]]
  approval surface — the console is where the filtered stream lands.)

## Capacity — can 10 sub-MOMOs run 10 projects from the Mac mini? (Eli, 2026-07-04)
Yes, with one thing understood + two real ceilings:
- **The inference is NOT on the mini.** Each MOMO's model runs on Anthropic's servers; the mini only does
  orchestration + hands-on work (edits, builds, tests). So 10 "brains" doesn't tax the mini itself.
- **Ceiling 1 — local build load.** 10 concurrent builds/test-suites/dev-servers IS heavy for a Mac mini.
  This is where Eli's M3 Max/128GB genuinely helps, OR offload heavy builds to cloud agents (known path —
  cloud routines already run).
- **Ceiling 2 — API rate limits + cost.** 10 MOMOs = 10× tokens → hits usage caps (this literally caused
  the 2026-07-03 model-limit blackout, see [[../goals/momo-engine-hardening-2026-07-03]]) + real $. Needs
  budgeting + staggering, not "launch 10 and go."
- Both solvable: cloud offload for builds, budget/rate management for API. 10 projects is realistic once
  those two are handled.

## Why this matters
This is the concrete shape of the "smart orchestration of agents" governing goal + the answer to Eli's
"if I had my M3 Max, would it be faster?" (real parallel building needs this layer). Sequenced AFTER the
NV deadline. Depends on / overlaps [[../goals/momo-engine-hardening-2026-07-03]] (scoped roles + egress +
verification layer are pieces 2 & 3). See also [[work-engine]], [[gsd-workengine-design]].
