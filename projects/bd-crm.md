# BD CRM — Business Development pipeline

Custom CRM for Eli's BD flow: research a target → enrich (practice + practitioners) →
score opportunity → audit their website → draft a better build → report to Eli, who
runs discovery + sales.

**Status:** v0 scaffolded and running locally (2026-06-22). Schema + migration + seed +
read UI all verified working.

## Decisions
- **New standalone system** (Eli confirmed) — NOT merged with the clinic OS CRM.
  Schema kept clean so it *could* merge later, but they're separate for now.
- **Stack:** Next.js 16 (App Router, Turbopack) + TypeScript, Tailwind v4, Prisma 7
  with the **pg driver adapter**, Postgres. Run with **bun** (no node/npm on the mini).
- **Dev DB:** local Postgres 16 (Homebrew, `brew services`), database `bd_crm`,
  user `momo`. Designed Azure-Postgres-ready — prod connection string just slots into
  `DATABASE_URL` / `prisma.config.ts` at deploy.

## Location & run
- Repo: `/Users/momo/momo/projects/bd-crm`
- Dev: `bun run dev` → http://localhost:3000 (LAN: 192.168.0.210:3000)
- `bun run db:migrate | db:seed | db:generate | db:studio`, `bun run typecheck`

## Prisma 7 gotchas (cost real time — remember these)
- `url` is **no longer allowed in schema.prisma**. Connection lives in
  `prisma.config.ts` under `datasource.url` (for migrate) — NOT a top-level `adapter`
  key (that's not in the config type). Runtime client builds its own `PrismaPg`
  adapter in `lib/prisma.ts`.
- Generator output is explicit: `output = "../lib/generated/prisma"`, imported via
  `@/lib/generated/prisma`. It's gitignored.
- `migrate dev` needs `datasource.url` in the config even though schema dropped it.
- bun blocks Prisma's postinstall → `bun pm trust --all` before `prisma generate`.
- Next.js: `pg`, `@prisma/adapter-pg`, `@prisma/client` must be in
  `serverExternalPackages` (next.config.ts) or Turbopack fails to load `pg` (500s).
- Next.js 16: page `params` is async (`await props.params`); `PageProps<'/route'>`
  global helper needs `next typegen` (or dev/build) to know the route.

## Data model (6 objects)
- **Practice** — the target business. status: NEW→RESEARCHING→ENRICHED→AUDITED→QUALIFIED→ARCHIVED.
- **Practitioner** — people at a practice (role, AHPRA #, primary contact flag).
- **Opportunity** — the deal. stage (NEW→…→WON/LOST), score 0-100 + rationale, est. value.
- **Audit** — website snapshot, many over time. Quality scores, detected stack +
  StackRisk + flags, legal checks (APP-5 collection notice / privacy / terms),
  homepage report. Mirrors the [[methodologies/website-audit]] checklist.
- **Activity** — append-only timeline (NOTE/RESEARCH/OUTREACH/STATUS_CHANGE/AUDIT_RUN/REPORT_GENERATED).
- **Report** — the deliverable handed to Eli (markdown content, recommendation, status).

All FK-linked with indexes on region/specialty/status/stage/score; cascade deletes
from Practice; timestamps everywhere.

## Next steps (not yet built)
- Write path beyond "add practice": add practitioners / opportunities / audits / reports
  from the UI (currently seed + research-flow only).
- Wire the BD research flow to write enriched targets straight into the DB
  (multi-agent batch → Practice + Practitioner + Opportunity + Audit rows).
- Auth before any deploy (server actions are currently unauthenticated).
- Deploy to Azure when Eli's ready (the cost gate).
