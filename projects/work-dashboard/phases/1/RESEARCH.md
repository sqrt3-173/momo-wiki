# RESEARCH.md — Phase 1: Projects at a Glance

> **Phase:** 1 — Projects at a Glance (work-dashboard) · **Researched:** 2026-07-02
> **Domain:** Local read-only Next.js dashboard over Postgres 17 (greenfield foundation)
> **Overall confidence:** HIGH (stack + read-only mechanism verified against registries and official docs; Bun-runtime risk assessment MEDIUM because issue statuses move per release)
> *Open questions Q1/Q2/Q4 were resolved by orchestrator psql probes 2026-07-02 — answers inlined below in italics.*

## User Constraints

**Locked decisions (from PROJECT.md D-01..D-05 + orchestrator):**
- Next.js/React + shadcn/ui (house stack), TypeScript, tests written.
- Local-only on the Mac mini, accessed from a MacBook on the home network. No internet exposure, no auth in v1 (D-01).
- Read-only: the app must be incapable of writing to `momo_work` (D-02) — dedicated read-only Postgres role as mechanism, best practice verified below.
- Data source: Postgres 17 (Homebrew) on localhost, database `momo_work`. Existing rollup views: `project_status`, `module_status`, `phase_status`; functions `human_queue()`, `gate_blocking()`.
- Foundation phase: keep architecture compatible with later drill-down, wave lanes, live agent feed (near-real-time), timeline, guard log — but do not plan those phases.

**Claude's discretion:** runtime strategy (Bun vs Node — researched, recommendation below), Postgres client library, refresh mechanism, test tooling, project structure.

**Deferred ideas:** none passed.

## Phase Requirements

| REQ-ID | Description | Findings that enable it |
|---|---|---|
| VIEW-01 | Eli sees all projects with status and progress (done/doing/todo rollups) | `project_status` view is the single query source (Don't Hand-Roll #1); SSR server component + `pg` typed query (Code Examples 1–3); force-dynamic + client auto-refresh satisfies "no stale seed data" (Pitfall #1, Code Example 4); `dashboard_ro` role satisfies "write fails by construction" (Architecture Patterns, Code Example 5, test in Example 6) |

## Architectural Responsibility Map

| Capability | Owning tier | Rationale |
|---|---|---|
| Progress rollups (done/doing/todo math) | **Database** (`project_status` view) | Rollups already exist as views; recomputing in JS invites drift from `momo_work` truth (success criterion 2 says "match what's in momo_work") |
| Data reads | **Frontend Server (SSR)** — async server components via `lib/queries.ts` | DB is localhost-only; credentials never reach the browser; no separate API tier needed in phase 1 |
| Read-only enforcement | **Database** (role privileges) | Must fail *by construction* — app-level discipline is not a construction guarantee |
| Refresh trigger | **Browser/Client** (interval → `router.refresh()`) | Server stays stateless; each refresh is a fresh SSR pass hitting the live DB |
| UI components/styling | **Browser/Client** (shadcn/ui + Tailwind, rendered via RSC/SSR) | House stack |
| API/Backend (route handlers) | **None in phase 1** — but keep queries in `lib/` so phase 4 SSE route handlers reuse them | Foundation compatibility without designing phase 4 |
| CDN/Static | **None** | Local-only; static export is explicitly rejected (kills live server reads) |

## Summary

Current stable is **Next.js 16.2.10** (Turbopack default for dev and build, React 19.2, minimum Node 20.9) [VERIFIED: npm registry + nextjs.org upgrade guide]. The phase's biggest question — can Bun 1.3.x replace Node as the runtime — resolves clearly: Bun officially documents `bun --bun next dev/build/start` and it broadly works, but the **Bun-runtime + Next 16 Turbopack combination has a live trail of crash-grade and dev-experience issues** through early 2026: CommonJS-wrapper build crashes (fixed post-1.3.5, but a regression pattern), excessive Fast Refresh rebuild loops (open, confirmed-tracked by the Next.js team as of Feb 2026), Cache Components `setTimeout` warnings, and module-resolution failures in Turbopack workers. The Next.js team does not test or officially support Bun as a runtime — `engines` says Node ≥ 20.9.0. Community consensus lands on: Bun as package manager is a safe win; Bun as production runtime only after validating your exact package set, with Node as the boring default.

For a foundation that five later phases build on — including phase 4's streaming/SSE needs, where runtime fidelity matters most — the right call is **one approval request to Eli: install Node.js 24 LTS** (Active LTS until Oct 2026, then maintenance to 2028), and keep Bun 1.3.14 as package manager (`bun install`, `bunx` — fast, already installed, officially supported by Next.js docs). Full-Bun is the documented fallback if Eli declines, with pinned Bun and accepted dev flakiness. For DB access, **`pg` (node-postgres) 8.22.0** with hand-typed row interfaces: pure JS, built-in pooling, the most battle-tested Postgres client in the ecosystem, and it runs on both Node and Bun so the fallback path doesn't change the data layer. No ORM — five read-only queries against existing views don't earn codegen ceremony.

Read-only-by-construction is a solved Postgres pattern: a dedicated `LOGIN` role granted only `CONNECT`/`USAGE`/`SELECT` (+ `EXECUTE` for the two functions), `ALTER DEFAULT PRIVILEGES` for future objects, and `default_transaction_read_only = on` as defense-in-depth (privileges are the boundary; the GUC is session-overridable and must not be treated as the guarantee). An integration test asserting an INSERT fails with SQLSTATE 42501/25006 makes success criterion 4 executable.

**Primary recommendation:** Ask Eli to approve `brew install node@24` (one install), then scaffold Next.js 16.2.x + Tailwind 4 + shadcn/ui with Bun as package manager, query `project_status` via `pg` from a force-dynamic server component with a 10s client `router.refresh()`, connecting as a new `dashboard_ro` Postgres role.

## Standard Stack

### Core

| Component | Version | Verified | Notes |
|---|---|---|---|
| Next.js | 16.2.10 | [VERIFIED: registry.npmjs.org/next/latest] | `engines: node >=20.9.0`; Turbopack default dev+build; webpack still available via `--webpack` [VERIFIED: nextjs.org/docs/app/guides/upgrading/version-16] |
| React / react-dom | 19.2.x (bundled canary line Next 16 uses) | [CITED: nextjs.org v16 upgrade guide] | Install `react@latest react-dom@latest` alongside next |
| Node.js | 24 LTS (**requires Eli's install approval**) | [VERIFIED: endoflife.date/nodejs — 24 is Active LTS, ends active Oct 20 2026 then maintenance] | Runtime for dev + `next start`. Node 26 is Current, not LTS until ~Oct 2026 — don't chase it |
| Bun | 1.3.14 (already installed) | [VERIFIED: orchestrator probe] | Package manager + script runner only (`bun install`, `bunx`). NOT the app runtime (see Pitfalls) |
| TypeScript | ≥ 5.1 (latest 5.x) | [CITED: Next 16 upgrade guide minimum 5.1.0] | |
| Tailwind CSS | 4.3.2 | [VERIFIED: registry.npmjs.org/tailwindcss/latest] | v4 CSS-first config (`@theme`), no tailwind.config.js by default |
| shadcn CLI | 4.12.0 (`bunx shadcn@latest`) | [VERIFIED: registry.npmjs.org/shadcn/latest] | `init` supports Next.js + Tailwind v4 + React 19 [CITED: ui.shadcn.com/docs/installation/next, /docs/tailwind-v4] |
| pg (node-postgres) | 8.22.0 | [VERIFIED: registry.npmjs.org/pg/latest] | + `@types/pg` dev dep. Do NOT install optional `pg-native` |
| Postgres | 17 (Homebrew, running) | [VERIFIED: orchestrator probe] | `momo_work` on localhost:5432 |

### Supporting

| Component | Version | Verified | Purpose |
|---|---|---|---|
| Vitest | 4.1.9 | [VERIFIED: registry.npmjs.org/vitest/latest] | Unit + DB integration tests (runs on Node) |
| @testing-library/react + jsdom + @vitejs/plugin-react | latest | [CITED: nextjs.org Vitest guide pattern] | Client-component tests. Note: async Server Components are NOT unit-testable in Vitest — test the query layer + client components instead [CITED: nextjs.org/docs/app/guides/testing/vitest] |
| shadcn MCP | — | [ASSUMED: may be installed by execution time] | Available tooling for the executor to browse/add components; do not depend on it — `bunx shadcn@latest add <component>` is the baseline path |

### Alternatives considered

| Option | Verdict | Why rejected |
|---|---|---|
| **Bun 1.3.14 as full runtime** (`bun --bun next ...`) | Fallback only | Open/recurring Bun+Turbopack issues (see Pitfalls #4); Next.js team doesn't support/test Bun runtime [VERIFIED: no maintainer support statement in vercel/next.js#55272; engines field] |
| Next 15.x under Bun (webpack era, more Bun mileage) | Rejected | Greenfield on N-1 major to accommodate a runtime workaround is a fragile foundation; 15.x is already maintenance-line |
| Static export + tiny Bun server | Rejected | No server rendering at request time → cannot serve live DB reads; kills phase 4 entirely |
| postgres.js 3.4.9 | Solid, not chosen | [VERIFIED version: npm] Nice tagged-template API, faster in microbenchmarks, but `pg` has broader adoption, more Next.js-specific prior art, and identical behavior across Node/Bun fallback |
| Drizzle / Kysely | Rejected for phase 1 | Type-safe codegen earns nothing for ~5 read-only SELECTs against pre-built views; hand-typed row interfaces are smaller and honest. Revisit only if the query surface grows past ~15 queries |
| `Bun.sql` (built-in Postgres client) | Rejected | Couples the data layer to the Bun runtime we're recommending against |
| pg-native | Rejected | C bindings — exactly the N-API class of thing that breaks under alternate runtimes; zero benefit at this scale |

### Install commands

```bash
# GATE: requires Eli's approval (install)
brew install node@24        # keg-only: needs PATH entry, see Assumptions A-1

# Scaffold (Bun as package manager — officially supported by Next.js docs)
bunx create-next-app@latest dashboard --typescript --tailwind --app --eslint
cd dashboard
bunx shadcn@latest init
bun add pg
bun add -d @types/pg vitest @vitejs/plugin-react @testing-library/react jsdom
```

## Package Legitimacy Audit

| Package | Registry | Age / adoption | Source repo | postinstall | Verdict |
|---|---|---|---|---|---|
| next 16.2.10 | npm | 2016+, top-10 package | vercel/next.js | none suspicious | **OK** |
| react / react-dom 19.2.x | npm | flagship | facebook/react | none | **OK** |
| pg 8.22.0 | npm | 2010+, ~10M wk downloads class | brianc/node-postgres | none (pg-native is optional & excluded) | **OK** |
| @types/pg | npm | DefinitelyTyped | DT monorepo | none | **OK** |
| tailwindcss 4.3.2 | npm | flagship | tailwindlabs/tailwindcss | none | **OK** |
| shadcn 4.12.0 (CLI, run via bunx, not a dependency) | npm | official shadcn/ui CLI | shadcn-ui/ui | CLI writes files by design — run in repo, review diff | **OK** |
| vitest 4.1.9 | npm | vitest-dev/vitest | — | none | **OK** |
| postgres 3.4.9 (not installed; noted for completeness) | npm | porsager/postgres [VERIFIED repo via registry] | — | — | OK (unused) |

No new/low-download/no-repo packages in this plan; nothing requires a slop checkpoint. The only human gate is the **Node 24 install** (guard's install rule), not package legitimacy.

## Architecture Patterns

### Data flow (VIEW-01, traceable input → output)

```
MacBook browser ──HTTP http://<mini>.local:3000──▶ next start (Node 24, Mac mini, -H 0.0.0.0)
                                                        │
                                       app/page.tsx (async Server Component,
                                       export const dynamic = "force-dynamic")
                                                        │
                                              lib/queries.ts (typed)
                                                        │
                                       lib/db.ts  pg.Pool (max 5, globalThis singleton)
                                                        │  TCP localhost:5432
                                                        ▼  role: dashboard_ro (SELECT-only)
                                     Postgres 17 · momo_work · view project_status
                                                        │
                              rows ──▶ <ProjectGrid> (shadcn Card/Badge/Progress) ──▶ HTML

<AutoRefresh> (client component) ── every 10s ──▶ router.refresh() ──▶ fresh SSR pass ──▶ fresh SELECT
```

### Recommended structure

```
dashboard/
  app/
    layout.tsx            # shell, <AutoRefresh /> mounted here or in page
    page.tsx              # projects-at-a-glance (force-dynamic)
    globals.css           # Tailwind v4 @theme + shadcn tokens
  components/
    ui/                   # shadcn-generated (never hand-edit conventions)
    project-grid.tsx      # presentation only — takes rows as props
    auto-refresh.tsx      # "use client" interval → router.refresh()
  lib/
    db.ts                 # pg.Pool singleton (ONLY place a connection is made)
    queries.ts            # one exported typed function per view/function
  tests/
    queries.test.ts       # integration: real local DB, RO role
    readonly.test.ts      # write attempts MUST fail (success criterion 4)
    project-grid.test.tsx # unit: rollup rendering
  .env.local              # DATABASE_URL (gitignored)
```

**Named patterns:**
- **DB-owned rollups** — server components render `SELECT * FROM project_status`; zero aggregation in JS. Comparable dashboards that recompute rollups client-side drift from the source of truth; the view IS the contract.
- **globalThis pool singleton** — standard Next.js pattern to survive dev HMR (source: node-postgres/Next.js community convention, same shape Prisma documents for its client) — Code Example 1.
- **SSR + client-initiated refresh** (`router.refresh()`) — the documented App Router way to re-render server components with fresh data without full page reload [CITED: nextjs.org/docs/app/api-reference/functions/use-router].
- **Upgrade path (note only, per constraint):** polling (now) → SSE route handler in `app/api/` reusing `lib/queries.ts` → optionally pg `LISTEN/NOTIFY` feeding that stream (phase 4). Nothing in this structure blocks it; Node runtime keeps streaming responses on the supported path. Do not build any of it now.

**Anti-patterns to avoid:** API routes wrapping the DB just so client components can fetch (unnecessary tier in phase 1); `cacheComponents: true` (new caching semantics + a known Bun warning; adds nothing to a force-dynamic dashboard); connection-per-request instead of a pool; putting `DATABASE_URL` anywhere `NEXT_PUBLIC_`.

## Don't Hand-Roll

| Problem | Don't build | Use instead | Why |
|---|---|---|---|
| done/doing/todo rollups | JS aggregation over raw tables | Existing `project_status` / `module_status` / `phase_status` views | They're already the engine's own definition of progress; criterion 2 is "matches momo_work" — the view can't disagree with itself |
| Connection management | ad-hoc `Client` per request, custom retry | `pg.Pool` | Handles queuing, idle reaping, error recovery; connection-per-request exhausts Postgres under refresh polling |
| Read-only enforcement | app-layer "only SELECT allowed" query filtering | Postgres role privileges (+ RO GUC as belt) | By-construction means the credential can't write even if the app is buggy or compromised |
| UI primitives (cards, badges, progress bars, tables) | custom CSS components | shadcn/ui via CLI | House stack; accessible; themed once via Tailwind v4 tokens |
| Data refresh | custom fetch + client state store | `router.refresh()` on an interval | Built into App Router; keeps a single rendering path (SSR) instead of two data paths |
| Project scaffold | manual webpack/tsconfig assembly | `create-next-app` + `shadcn init` | Preflight-checked, Tailwind v4 wired correctly [CITED: ui.shadcn.com/docs/installation/next] |

**Key insight:** phase 1 has almost no novel logic — its entire job is wiring proven pieces so later phases inherit a clean seam (`lib/queries.ts`) and a credential that cannot write.

## Common Pitfalls

1. **Static prerender serves stale data (breaks success criterion 3).** A server component doing a `pg` query (not `fetch`) gets prerendered at build time by default — the page ships build-time rows forever. *Avoid:* `export const dynamic = "force-dynamic"` on the page (or `await connection()`). *Warning sign:* refresh doesn't change data after a DB update; build log shows the route as `○ (Static)` instead of `ƒ (Dynamic)`. [CITED: nextjs.org rendering docs; the v16 guide's `connection()` note]
2. **Dev HMR leaks connection pools.** Every hot reload re-evaluates modules; a bare `new Pool()` piles up connections until Postgres refuses. *Avoid:* globalThis singleton (Code Example 1). *Warning sign:* `remaining connection slots are reserved` after a dev session.
3. **Treating `default_transaction_read_only` as the security boundary.** It's a session GUC — any connection can `SET transaction_read_only = off`. The boundary is *absence of INSERT/UPDATE/DELETE/TRUNCATE privileges*; the GUC is defense-in-depth only. [CITED: postgresql.org docs; Crunchy Data read-only-user post]
4. **Bun-as-runtime assumed safe because `bun --bun next dev` starts.** Documented failure modes with Next 16 + Turbopack: CommonJS-wrapper crash during build SSG phase (oven-sh/bun #25609 — closed with fix post-1.3.5; sibling #25650 same class), excessive Fast Refresh rebuild loops (vercel/next.js #89530 — **open**, confirmed-tracked, Feb 2026), Cache Components `setTimeout` runtime warning (oven-sh/bun #25639), Turbopack-worker module resolution failures (oven-sh/bun #24419, vercel/next.js #86866). *Avoid:* Node 24 runtime. *If fallback taken:* pin Bun, keep `cacheComponents` off, re-verify `bun --bun next build` output after every Bun upgrade.
5. **`ALTER DEFAULT PRIVILEGES` run as the wrong role.** It only applies to objects *created by the role that ran it* — if the engine's owner role creates a new view later, `dashboard_ro` silently can't read it. *Avoid:* run it as (or `FOR ROLE`) the actual owner of `momo_work` objects. *Resolved: all objects owned by `momo` (orchestrator probe) — run the statements as `momo`.* *Warning sign:* new view 404s with `permission denied` only for the dashboard. [VERIFIED: postgresql.org ALTER DEFAULT PRIVILEGES docs + Crunchy Data]
6. **Server binds localhost only → MacBook gets connection refused.** Pass `-H 0.0.0.0` explicitly to `next start`/`next dev`; expect a one-time macOS firewall allow prompt for the node binary (that prompt is not a system-settings change).
7. **Next 16 removed/renamed things training data still suggests:** `next lint` is gone (use ESLint directly), `middleware` → `proxy`, sync `params`/`searchParams`/`cookies` access fully removed — all request APIs are async-only. [VERIFIED: v16 upgrade guide]
8. **Serverless-era advice misapplied:** "you must run PgBouncer in front of Postgres" is for serverless fan-out. One long-lived Node process on the same machine needs a `max: 5` pool and nothing else.
9. **Secrets hygiene even though local:** `DATABASE_URL` in `.env.local` (gitignored), never `NEXT_PUBLIC_`, never imported into a `"use client"` file.

## Code Examples

**1. `lib/db.ts` — pool singleton** (pattern: node-postgres docs + standard Next.js HMR-safe convention)
```ts
import { Pool } from "pg";

const globalForPg = globalThis as unknown as { pgPool?: Pool };

export const pool =
  globalForPg.pgPool ??
  new Pool({
    connectionString: process.env.DATABASE_URL, // postgres://dashboard_ro:***@localhost:5432/momo_work
    max: 5,
  });

if (process.env.NODE_ENV !== "production") globalForPg.pgPool = pool;
```

**2. `lib/queries.ts` — typed view query** (row shape confirmed against the live view by orchestrator probe 2026-07-02)
```ts
import { pool } from "./db";

export interface ProjectStatusRow {
  id: number;
  slug: string;
  name: string;
  status: string;
  priority: number;
  total_tasks: string;   // bigint comes back as string from pg — parse or cast in SQL
  done_tasks: string;
  doing_tasks: string;
  todo_tasks: string;
  blocked_tasks: string;
}

export async function getProjectStatuses(): Promise<ProjectStatusRow[]> {
  const { rows } = await pool.query<ProjectStatusRow>(
    "SELECT * FROM project_status ORDER BY slug",
  );
  return rows;
}
```

**3. `app/page.tsx` — dynamic SSR page** (pattern: Next.js App Router data fetching docs)
```tsx
import { getProjectStatuses } from "@/lib/queries";
import { ProjectGrid } from "@/components/project-grid";

export const dynamic = "force-dynamic"; // never prerender — every request hits the live DB

export default async function DashboardPage() {
  const projects = await getProjectStatuses();
  return <ProjectGrid projects={projects} />;
}
```

**4. `components/auto-refresh.tsx` — refresh loop** (pattern: `useRouter().refresh()`, nextjs.org use-router reference)
```tsx
"use client";
import { useRouter } from "next/navigation";
import { useEffect } from "react";

export function AutoRefresh({ intervalMs = 10_000 }: { intervalMs?: number }) {
  const router = useRouter();
  useEffect(() => {
    const id = setInterval(() => router.refresh(), intervalMs);
    return () => clearInterval(id);
  }, [router, intervalMs]);
  return null;
}
```

**5. Read-only role — run once against `momo_work`** (pattern: Crunchy Data "Creating a Read-Only Postgres User" + postgresql.org ALTER DEFAULT PRIVILEGES; adapted to this DB; run as `momo`, the owner of all objects)
```sql
CREATE ROLE dashboard_ro LOGIN PASSWORD '<generate>' NOSUPERUSER NOCREATEDB NOCREATEROLE;
GRANT CONNECT ON DATABASE momo_work TO dashboard_ro;
GRANT USAGE ON SCHEMA public TO dashboard_ro;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO dashboard_ro;      -- includes views
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO dashboard_ro;  -- human_queue(), gate_blocking()

-- Future objects: run as momo (owner of all momo_work objects — probe-confirmed)
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO dashboard_ro;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT EXECUTE ON FUNCTIONS TO dashboard_ro;

-- Defense in depth only — NOT the boundary (session-overridable):
ALTER ROLE dashboard_ro SET default_transaction_read_only = on;
```
**CAUTION:** `GRANT EXECUTE ON ALL FUNCTIONS` includes the engine's mutating functions (`claim_next_plan`, `create_gate`, `materialize_*`) — but those all write to tables, and `dashboard_ro` has no table write privileges, so calls fail at the table layer (SECURITY INVOKER, probe-confirmed). Tighter alternative: grant EXECUTE only on `human_queue`/`gate_blocking` and skip the blanket function grant + its default privilege. **Prefer the tighter form.**

**6. `tests/readonly.test.ts` — success criterion 4 as a test** (Vitest)
```ts
import { expect, it } from "vitest";
import { pool } from "@/lib/db";

it("dashboard credentials cannot write to momo_work", async () => {
  await expect(
    pool.query("INSERT INTO projects (slug) VALUES ('should-never-land')"),
  ).rejects.toMatchObject({
    code: expect.stringMatching(/^(42501|25006)$/), // permission denied | read-only transaction
  });
});
```

## State of the Art

| Old approach | Current approach | Changed |
|---|---|---|
| Next.js 15, webpack default | Next.js 16, Turbopack default dev+build (`--webpack` to opt out) | Oct 2025 (v16) [VERIFIED: upgrade guide] |
| `next lint` | Removed — ESLint/Biome directly; `next build` no longer lints | v16 |
| `middleware.ts` | `proxy.ts` (Node runtime only) | v16 (deprecated name) |
| Sync `params`/`searchParams`/`cookies` | Async-only (`await props.params`) | v15 introduced, v16 removed sync compat |
| `unstable_cacheLife`/`unstable_cacheTag`, PPR flags | `cacheLife`/`cacheTag` stable; `cacheComponents` config (we keep it OFF) | v16 |
| Tailwind v3 `tailwind.config.js` | Tailwind v4 CSS-first `@theme` | 2025; shadcn CLI initializes v4 [CITED: ui.shadcn.com/docs/tailwind-v4] |
| `npx shadcn-ui@latest` | `npx/bunx shadcn@latest` (CLI 4.x) | CLI renamed; 4.12.0 current [VERIFIED: npm] |
| "Bun can't run the Next dev server" (2023) | `bun --bun next dev/build/start` works nominally; Vercel Functions gained a Bun runtime (Nov 2025) — but self-hosted Next-on-Bun remains unsupported by the Next team and buggy with Turbopack | 2023 → 2026 [CITED: bun.com guide; vercel/next.js#55272] |
| Node 22 LTS default | Node 24 Active LTS (26 becomes LTS ~Oct 2026) | Oct 2025 [VERIFIED: endoflife.date] |
| Read-only users via manual per-table grants | Explicit grants + `ALTER DEFAULT PRIVILEGES`; `pg_read_all_data` (PG14+) as the coarse-grained alternative | PG14+ [VERIFIED: postgresql.org / Crunchy Data] |

## Assumptions Log

| # | Assumption | Section | Risk if wrong |
|---|---|---|---|
| A-1 | `node@24` Homebrew formula exists and is keg-only (needs `export PATH="$(brew --prefix node@24)/bin:$PATH"` or symlink) | Install commands | Low — worst case `brew install node` (26 Current) or a version manager; adjust the approval request wording |
| A-2 | Mac mini reachable from the MacBook as `momo-mini.local` (mDNS) | Data flow | Low — fall back to LAN IP; doesn't affect architecture |
| A-3 | `next start`/`next dev` do not bind LAN interfaces by default in all configs — hence explicit `-H 0.0.0.0` | Pitfall #6 | None if wrong (explicit flag is harmless) |
| A-4 | `pg` (pure JS) runs correctly under Bun 1.3.x (fallback path only) | Alternatives | Medium for the fallback path only; the recommended Node path is unaffected. Verify with one smoke query if fallback is taken |
| A-5 | ~~`momo_work` objects live in schema `public`; functions are read-only SECURITY INVOKER~~ **RESOLVED by probe:** all objects in `public` owned by `momo`; `human_queue`/`gate_blocking` are SECURITY INVOKER (prosecdef=f) | Code Example 5 | — |
| A-6 | shadcn MCP may be installed by execution time | Supporting stack | None — `bunx shadcn@latest add` is the baseline either way |
| A-7 | Port 3000 free on the mini | Data flow | Trivial — pick another port |

## Open Questions

| Q | Status |
|---|---|
| Q1 — which role owns `momo_work` objects | **RESOLVED (probe 2026-07-02):** `momo` owns everything in `public`; run default-privileges statements as `momo` |
| Q2 — exact `project_status` columns | **RESOLVED (probe):** id int, slug text, name text, status text, priority int, total_tasks/done_tasks/doing_tasks/todo_tasks/blocked_tasks bigint — interface pinned in Code Example 2 (note: `pg` returns bigint as string) |
| Q3 — Node 24 install approval | **Human checkpoint, front-loaded:** asked of Eli via Discord 2026-07-02 alongside the shadcn installs. Fallback (full-Bun, pinned, risks documented) is the standing option 2 |
| Q4 — functions read-only SECURITY INVOKER | **RESOLVED (probe):** both SECURITY INVOKER; with no table-write privileges the RO boundary holds |

## Environment Availability

| Dependency | Status | Detail |
|---|---|---|
| Bun 1.3.14 | **Available** | Battle-tested on this machine (Discord bridge) [VERIFIED: orchestrator probe] |
| Node.js | **Missing — blocking for recommended path** | Gated on Eli's install approval; fallback = full-Bun runtime (documented risks) |
| Postgres 17 + `momo_work` | **Available** | Homebrew, localhost [VERIFIED: orchestrator probe] |
| psql (read-only, guard-permitted) | **Available** | Used for the probes above |
| Internet access for `bun install` | Available (egress proxy) | Registry fetches only |

**Missing-blocking list:** Node 24 LTS (pending one approval). Nothing else blocks.

## Sources

**Primary (HIGH confidence):**
- registry.npmjs.org — `next` 16.2.10 (`engines >=20.9.0`), `pg` 8.22.0, `postgres` 3.4.9, `tailwindcss` 4.3.2, `shadcn` 4.12.0, `vitest` 4.1.9 (fetched this session)
- nextjs.org/docs/app/guides/upgrading/version-16 — Node/TS minimums, Turbopack default, webpack `--webpack`, async APIs, removals (fetched this session, lastUpdated 2026-05-13)
- endoflife.date/nodejs — Node 24 Active LTS, Node 26 Current
- github.com/oven-sh/bun/issues/25609 (closed, fix PR #25710), github.com/vercel/next.js/issues/89530 (open, confirmed-tracked) — fetched this session

**Secondary (MEDIUM):**
- bun.com/docs/guides/ecosystem/nextjs — official `bun --bun` guide (no caveats enumerated — treated as marketing-adjacent)
- ui.shadcn.com/docs/installation/next, /docs/tailwind-v4, /docs/cli
- postgresql.org/docs/current/sql-alterdefaultprivileges.html; crunchydata.com/blog/creating-a-read-only-postgres-user
- github.com/vercel/next.js/discussions/55272 — absence of official Bun-runtime support statement
- github.com issue cluster: oven-sh/bun #25639, #25650, #24419, #25864; vercel/next.js #86866

**Tertiary (LOW — directional only):** dev.to / community posts on Bun-vs-Node production consensus and pg-vs-postgres.js benchmarks.

## Metadata

| Area | Confidence | Reason |
|---|---|---|
| Versions (next, pg, tailwind, shadcn, vitest, Node LTS) | HIGH | Registry-verified this session |
| Runtime recommendation (Node 24 over Bun) | HIGH | Multiple corroborating primary sources; open confirmed issue on the exact Bun+Turbopack combo |
| Bun issue statuses | MEDIUM | Moves per release — re-check if execution slips past ~1 week |
| Read-only role mechanism | HIGH | Official PG docs + Crunchy; caveats explicitly captured; ownership probe-confirmed |
| `momo_work` specifics (schema, view shapes, function security) | HIGH | Probe-resolved 2026-07-02 |
| Refresh model + upgrade path | HIGH | Standard App Router pattern; upgrade path noted without design |

**Research date:** 2026-07-02 · **Valid until:** 2026-08-01 (general stack) / **2026-07-09** for Bun↔Next compatibility claims and exact patch versions (fast-moving).
