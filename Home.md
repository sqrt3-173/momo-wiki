# MOMO Wiki — Home

MOMO's institutional memory — Eli's businesses, decisions, and methods. Synced via git between the
Mac mini (MOMO) and Eli's MacBook (Obsidian). Read at session start; kept current as work happens
(see [[skills/wiki-capture/SKILL|wiki-capture]]).

> **NUNU is retired (2026-07-02).** Eli: it's irrelevant now — never mention it or propose NUNU
> work. NUNU pages ([[ops/multi-instance]], [[ops/nunu-setup-blueprint]]) are historical record only.

> If you're reading this on the MacBook and it appeared after a **Git: Pull**, the two-way sync works. 🎉

## Map
- **goals/** — the living goals layer MOMO's triage reasons from: [[goals/nv-health]] · [[goals/skip-bin]] · [[goals/momo-engine]]. **Eli: edit these directly** — they're the machine's sense of what matters; saying "goal: ..." once on Discord also updates them.
- **[[projects/bariatric-intelligence]]** — the bariatric BD market intelligence (surgeons, clinics, allied health, fees, politics map).
- **[[projects/skip-bin/PROJECT|skip-bin]]** — the skip-bin booking business: [[projects/skip-bin/PROJECT|PROJECT]] · [[projects/skip-bin/REQUIREMENTS|REQUIREMENTS]] · [[projects/skip-bin/ROADMAP|ROADMAP]] (rendered from `momo_work` — don't hand-edit). **Parked** — `confirm_roadmap` gate open, nothing runs.
- **[[projects/work-dashboard/PROJECT|work-dashboard]]** — mission control for the work engine (local, read-only v1): [[projects/work-dashboard/PROJECT|PROJECT]] · [[projects/work-dashboard/REQUIREMENTS|REQUIREMENTS]] · [[projects/work-dashboard/ROADMAP|ROADMAP]] (rendered — don't hand-edit). **v1 complete 2026-07-03** + [[projects/work-dashboard/DECISIONS|DECISIONS]] log.
- **[[projects/ingestion-loop/PROJECT|ingestion-loop]]** — the always-on layer (inbox → atoms → reasoning triage → seeds → heartbeat): [[projects/ingestion-loop/PROJECT|PROJECT]] · [[projects/ingestion-loop/REQUIREMENTS|REQUIREMENTS]] · [[projects/ingestion-loop/ROADMAP|ROADMAP]] · design spec [[projects/ingestion-loop-design]].
- **ops/** — how the machine + agent run:
  - [[ops/multi-instance]] — running MOMO + NUNU in parallel; hardware capacity; local sharing.
  - [[ops/nunu-setup-blueprint]] — the full second-instance + memory-architecture build plan + status.
  - [[ops/discord-bridge]] — the Discord channel: access, gotchas (always use the reply tool!).
  - [[ops/guardrails]] — the PreToolUse guard: what's allowed, what asks, the honest limits.
- **skills/** — procedural memory (playbooks MOMO pulls from):
  - [[skills/README|skill library index]] · [[skills/bd-research-sweep/SKILL|bd-research-sweep]] · [[skills/website-audit/SKILL|website-audit]] · [[skills/entity-enrichment/SKILL|entity-enrichment]] · [[skills/council-review/SKILL|council-review]] · [[skills/wiki-capture/SKILL|wiki-capture]]

## How to use
- **MOMO:** read relevant notes before working; capture learnings on completion; commit + push (wiki pushes are guard-auto-approved). *(NUNU runs the same playbooks but writes to its own `nunu-wiki`.)*
- **Eli (MacBook/Obsidian):** edit freely — the Obsidian Git plugin auto-pulls my changes and pushes yours every ~10 min.

(END)