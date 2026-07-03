# MOMO skill library — procedural memory

Each skill is a repeatable, proven methodology MOMO (and NUNU) pulls from to do work
consistently. This is the **procedural** half of the memory architecture; the **wiki** is the
semantic half (what's known). Skills encode *how*; the wiki encodes *what*.

## Why this exists
A methodology that lives only in one session dies with it. Skills make our best ways of working
durable, shareable across instances, and improvable over time — every run that teaches us something
updates the skill, so the next run starts smarter.

## Contract — every skill is a folder with a `SKILL.md`
Frontmatter:
```
---
name: <kebab-case>
description: <one line — when to reach for this>
---
```
Body sections (keep tight, operating-playbook style — not an essay):
- **When to use** — the trigger.
- **Inputs** — what you need before starting.
- **Method** — the ordered steps. Reference real scripts/files by path where they exist.
- **Output contract** — exactly what the skill produces.
- **Gotchas** — hard-won failure modes (the most valuable part; append every time we learn one).

## How they're used
- **Now:** MOMO reads the relevant `SKILL.md` before doing that kind of work (procedural memory).
- **Later (follow-up):** symlink `wiki/skills/*` into `.claude/skills/` so they're invocable via the
  Skill tool directly. Kept canonical here so they version + sync via git to NUNU + the MacBook.

## Current skills
- [`wiki-capture`](wiki-capture/SKILL.md) — capture learnings into the wiki at end of every job.
- [`website-audit`](website-audit/SKILL.md) — legal/security/quality audit of a target's site.
- [`bd-research-sweep`](bd-research-sweep/SKILL.md) — research targets → enrich CRM → score → report.
- [`entity-enrichment`](entity-enrichment/SKILL.md) — agent-wave deep enrichment of graph entities.
- [`council-review`](council-review/SKILL.md) — multi-agent (and later multi-model) adversarial review.
- [`ahpra-marketing-compliance`](ahpra-marketing-compliance/SKILL.md) — audit + reword medical/health marketing copy against AHPRA s133, the cosmetic-surgery guidelines, and the super-release warning; produces compliant copy + a human sign-off doc. Use on any health-client page/ad, and in BD audits of medical prospects.

## Adding/updating a skill
Build only from **proven** methodology, never speculation. When a run surfaces a new failure mode,
append it to that skill's Gotchas. Prune skills that stop being used. Commit with a clear message.
