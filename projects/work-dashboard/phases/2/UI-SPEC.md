---
phase: 2
slug: drill-down-wave-lanes
status: approved
shadcn_initialized: true
preset: default (nova-style tokens from scaffold)
created: 2026-07-02
---

# Phase 2 — UI Design Contract *(design system locked project-wide; interaction notes per phase)*

> GSD UI-SPEC (template: `worksystem/gsd-source/templates/UI-SPEC.md`). Compact ui-phase per DL-11:
> spec authored by orchestrator from the live scaffold's actual tokens; verification folded into the
> plan-checker's dimensions. Later phases add their interaction sections here or in their own spec.

## Design System

| Property | Value |
|----------|-------|
| Tool | shadcn (CLI 4.12, Tailwind v4 CSS-first) |
| Preset | scaffold default tokens (no custom preset) |
| Component library | base-ui (`@base-ui/react` — scaffold's base; check `base` field before assuming radix APIs) |
| Icon library | lucide-react |
| Font | Geist (sans) / Geist Mono (numbers, ids) |

## Spacing Scale

Tailwind defaults, multiples of 4. `gap-*` always (never `space-y-*`, per shadcn skill rules):
xs=4 icon gaps · sm=8 intra-card · md=16 card grids/default · lg=24 section padding · xl=32 page gutters.
Exceptions: none.

## Typography

| Role | Size | Weight | Notes |
|------|------|--------|-------|
| Body | 14px (`text-sm`) | 400 | default; dashboards run dense |
| Label/meta | 12px (`text-xs`) | 400–500 | slugs, timestamps, counts — `tabular-nums` for all numbers |
| Heading | 18px (`text-lg`) | 600 | card/section titles |
| Display | 24px (`text-2xl`) | 600 | page title only, one per page |

## Color — semantic tokens ONLY (never raw hex/named Tailwind colors)

| Role | Token | Usage |
|------|-------|-------|
| Dominant (60%) | `bg-background` | page |
| Secondary (30%) | `bg-card` / `bg-muted` | cards, lanes, rails |
| Accent (10%) | `bg-primary` / `text-primary` | **reserved for:** progress fill, active/doing states, links. Nothing else. |
| Destructive | `destructive` tokens | blocked states + failures only |

**Status mapping — LOCKED across every view (categorical consistency):**
`done` → muted-foreground text + filled progress · `doing` → primary · `todo` → muted · `blocked` → destructive · `cancelled` → muted + strikethrough.
The same status must never change color between screens.

## Copywriting Contract

| Element | Copy |
|---------|------|
| Empty project detail | "No phases yet" + "This project hasn't been roadmapped." |
| Empty wave lanes | "No plans yet" + "This phase hasn't been planned." |
| Error state (DB unreachable) | "Can't reach the work engine" + "Postgres on the mini isn't answering — check the machine." |
| Back navigation | "← All projects" |
| Blocked-by line | "waiting on {plan number(s)}" |

No CTAs in v1 — the app is read-only; never render buttons that imply actions.

## Phase 2 Interaction Contract

- **Routes:** `/` (exists) → `/projects/[slug]` (drill-down). Wave lanes render inside the project page per phase — no third level of navigation; depth stays ≤2 (dataviz: don't make the user hold a map).
- **Drill-down hierarchy:** milestone header → phases as sections (stage badge + REQ-IDs + progress) → each phase's plans as **wave lanes**: columns labeled "Wave N" left→right, plan cards colored by the locked status mapping, each card showing number, title, task done/total, and its "waiting on" line when deps are incomplete.
- **Density rule (dataviz):** a phase section must fit one laptop screen; plan cards are compact (title + 2 metadata lines max). Task lists collapse behind a native `<details>` disclosure per plan card — no dialogs for v1 read-only.
- **REQ coverage display:** REQ-IDs as small mono badges on each phase; orphaned (uncovered) REQs would show destructive-outline — data comes from the phase row.
- Project cards on `/` become links; hover = `bg-muted` transition only (no lift/shadow animation noise).

## Registry Safety

| Registry | Blocks Used | Safety Gate |
|----------|-------------|-------------|
| shadcn official | card, badge, progress, separator, breadcrumb (as needed) | not required |
| third-party | none | n/a |

## Checker Sign-Off

Folded into plan-checker dimensions (DL-11): copy ✓ (contract above) · visuals ✓ (shadcn primitives only) ·
color ✓ (semantic tokens, locked status map) · typography ✓ (4 roles) · spacing ✓ (gap-only, defaults) ·
registry ✓ (official only). **Approval: approved 2026-07-02 (autonomous mode, DL-08).**
