# Ingestion + Autonomous Loop — agreed design (pre-build)

> Captured from the Discord design session with Eli, 2026-07-03 (~08:35–09:00). This is the
> input spec for the project when Eli says go. Supersedes anything narrower said earlier.

## The system in one line
Always-on engine: Eli dumps anything, the machine decomposes it, reasons about it against
his businesses' goals, and either files knowledge, advances/creates projects, parks seeds —
with receipts for everything and his gates intact.

## Ingestion (Karpathy-style capture, made to bite back)
- **Inbox items are raw dumps, IMMUTABLE, never destroyed** — text notes, voice-memo
  transcripts, diagrams/images, videos-to-watch, files. Typed payloads; per-type extraction
  handlers; new input type = new handler, same model.
- **Decompose, then dispose:** ingestion extracts ATOMS (smallest self-contained thoughts).
  One memo → many atoms of different kinds. NEVER one-fate-per-item (Eli explicitly killed that).
- **Atom kinds/fates:** knowledge → wiki, filed by subject + linked · project idea → pipeline
  (requirements → roadmap → Eli's gate) · task → existing project, lane by blast radius
  (plain task vs planned change) · **seed** → parked with review date.
- **Provenance always:** every atom links to its source item; items link to their atoms.
  Old items can be re-ingested when context changes.

## Seeds + weekly review (the compounding layer)
- Seed = real idea, not viable yet (no home / missing prerequisite / maybe-someday).
- Weekly review is a **synthesis pass, never per-seed**: cluster all live seeds + recent
  knowledge atoms; overlapping seeds = one project candidate or a change-set to an existing
  project. Critical-mass clusters get pre-work (research, fit vs existing systems, mini-brief).
- Output: shaped weekly report to Eli — numbered decisions (start / merge-park / kill),
  change-sets against existing projects, retirements WITH reasons. Ambiguity → sharp questions,
  never guesses.

## Triage thinking: GOALS AS CONTEXT, NO RULES ENGINE (Eli's key correction)
- No standing-intents rules list. Instead: a **living goals layer** in the wiki per business
  (what it's for, direction, constraints, economics). Reasoning triage evaluates every atom:
  does this matter for where the businesses are going, how much (magnitude judgment), and
  what's the highest-leverage response — derived, not pattern-matched.
- **A stated goal is itself an input, the most powerful kind** — "double NV Health sales" /
  "expand into VIC + SA" gets DRILLED: decompose into drivers → pull known facts → locate the
  binding constraint → research gaps → strategy/expansion brief + proposed roadmap changes at
  Eli's gate.
- Judgment quality tracks context quality — goals live in the wiki so Eli can correct them;
  one Discord sentence updates the system's sense of what matters.

## The heartbeat loop
- Scheduled tick (~30 min): cheap DB check first (ready plans? untriaged inbox? seeds due?) —
  exits near-free when idle; wakes a fresh work session only when there's work. Same pipeline
  discipline (checker, tests, telemetry, digest). "MOMO pause"/"resume" via Discord.
- Dashboard: inbox panel (items → atoms → fates), loop status on the home strip, every triage
  decision + autonomous run in the digest.

## Non-negotiable boundaries (authority, not intelligence)
- Autonomous: research, quantify, cross-reference, draft, file, advance gated-open work.
- NEVER autonomous: contacting people, promising, spending, publishing, installing, deleting;
  Eli's gates stop what they stop today. Parallel executor fan-out = separate guard decision.

## Deliberately out of scope for v1
Parallel fan-out (guard decision pending); auto-approval of any hard-gated category.
