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

## Phase 3 Interaction Contract (added 2026-07-02 — same design system)

- **Route:** `/queue` — linked from the home header ("Your queue"), back link "← All projects". Depth stays ≤2.
- **Content:** Eli's open gates + human tasks (from `human_queue('eli')`), ranked by blocking count desc then priority. Each item: title, project (mono), priority, a blocking count badge ("blocks N"), and gate name (mono badge) when it's a gate.
- **Expansion (QUEUE-02):** native `<details>` per item listing the downstream tasks it holds up (title + project). Gate items additionally state what the gate withholds: "gate: {project} can't advance past {gate_name}". Zero-blocking items say "blocks nothing yet".
- **Empty state:** "Queue's clear" + "Nothing is waiting on you."
- **Color:** blocking count > 0 → primary emphasis; nothing destructive (waiting on a human is normal state, not failure). Status mapping unchanged.

## Phase 4 Interaction Contract (added 2026-07-02 — same design system)

- **Route:** `/agents` — "Agents" link in the home header next to "Your queue".
- **Live section (AGENT-01):** agents running NOW: agent kind (mono), host badge, project + plan/task
  title, started-at as relative time ("4m"), live = `ended_at IS NULL AND heartbeat within 3 min`;
  stale runs (no heartbeat, not ended) render muted with "stale" badge — honest, not hidden.
- **Recent runs strip:** last 10 finished runs: kind, outcome badge (ok → muted, error → destructive),
  duration, tokens if recorded. Full history/spend is phase 5 — do NOT build filters/pagination here.
- **Empty state:** "Nothing running" + "The engine is idle."
- **Times:** relative in the UI, tabular-nums; never fake precision (heartbeat granularity is minutes).

## Phase 5 Interaction Contract (added 2026-07-02 — same design system)

- **Extends `/agents`** (no new top-level route for history): live runs become expandable —
  native `<details>` per run revealing its breadcrumb feed (AGENT-02): last ~20 events, ts relative,
  kind mono, message plain. Recent strip grows into a **history section** on the same page: last 50
  runs (AGENT-03) — still no pagination; 50 is the v1 depth, "older history exists in the DB" noted.
- **Route `/digest`** (DIG-01): "Digest" home-header link. Chronological engine timeline, newest
  first: run completions, gates cleared, plans done, commits… — sourced from what the DB actually
  records (agent_events + tasks/plans updated_at); never invent event types with no source. Window
  filter via preset LINKS (`?window=overnight|24h|7d` — links are read-only navigation, not CTAs;
  overnight = 22:00–08:00 local of the most recent night). Each entry: relative time, project mono,
  one-line description.
- **Spend (USE-02/03):** a compact spend section on `/digest` (spend is a digest concern, and
  charts stay near their narrative): per-project totals for the window + per-run costs where
  recorded. **NULL costs excluded from sums with an explicit "N runs unpriced" note — sums of
  partial data must say so** (DL-16/DL-17 honesty). Simple bar-style rows (CSS widths on semantic
  tokens), NOT a charting library — v1 scale doesn't earn one; dataviz discipline applies.
- **Empty digest:** "Quiet window" + "Nothing happened in this window."

## Phase 6 Interaction Contract (added 2026-07-03 — same design system)

- **Route `/system`** — "System" home-header link. Two sections: Guard and Health.
- **Guard (GUARD-01):** last 100 guard decisions from `ops/logs/guard.log` (format:
  `<ts> ALLOW|BLOCK <tool> :: <reason>`), newest first. Per row: relative time, ALLOW (muted) /
  BLOCK (destructive-outline — a block is the guard working, but visually distinct) badge, tool
  mono, reason truncated. Filter = plain links `?show=all|allow|block`. Unreadable/absent log →
  "Guard log unavailable" + path shown (honest, not empty-state).
- **Health (HLTH-01):** best-effort panel, one row per check: DB reachable (SELECT 1), Discord
  bridge process present, work-loop heartbeat (latest agent_runs.heartbeat_at recency), disk free.
  States: ok (muted) / degraded (destructive-outline) / unknown (muted "?" — a failed CHECK is
  unknown, never assumed ok). **Verbatim caveat line on the panel:** "This panel shares the mini's
  fate — if the machine is down, so is this page. Real alerting must live elsewhere (D-05)."
- No auto-refresh exemption: same force-dynamic + 10s AutoRefresh.

## Registry Safety

| Registry | Blocks Used | Safety Gate |
|----------|-------------|-------------|
| shadcn official | card, badge, progress, separator, breadcrumb (as needed) | not required |
| third-party | none | n/a |

## Checker Sign-Off

Folded into plan-checker dimensions (DL-11): copy ✓ (contract above) · visuals ✓ (shadcn primitives only) ·
color ✓ (semantic tokens, locked status map) · typography ✓ (4 roles) · spacing ✓ (gap-only, defaults) ·
registry ✓ (official only). **Approval: approved 2026-07-02 (autonomous mode, DL-08).**
