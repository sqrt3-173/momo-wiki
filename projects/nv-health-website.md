# NV Health Website — Sydney landing + booking (DEADLINE: Sunday 2026-07-06 10:00 AEST)

**The deadline is real and Eli-stated (2026-07-03): EVERYTHING live at syd.nvhealth.com.au by
Sunday 10am** — landing page, instrumentation, working custom booking flow (his "1b — everything").

## Where things live
- Repo: `/Users/momo/momo/projects/nv-health-website` (clone of github.com/sqrt3-173/nv-health-website;
  auth via the shared fine-grained PAT — Eli extended it to this repo 2026-07-03; credential helper wired,
  no token on disk).
- Source of truth for project state: the repo's own GSD suite in `.planning/` (PROJECT, REQUIREMENTS,
  ROADMAP, STATE, research/, phases/). MOMO drives it with GSD-native artifacts — do NOT fork state into
  momo_work beyond the project row (id 8, priority 5).
- Design source of truth: `design-reference/ui_kits/website-fluid/Surgery Landing - Sydney v4.html`
  (+ BookingFormSydney.jsx). DS = `@nvhealth/design-system` workspace package (phase 1 productionised it).

## Position (2026-07-03 ~22:55)
- Phase 1/6 done + verified (FND-01..06).
- **Phases 2 + 3 PLANNED (checker-quality, Opus) and committed to the repo** (`.planning/phases/02-*/02-01..04-PLAN.md`
  all written; `.planning/phases/03-*/03-STATUS.md` holds the 5-plan structure + frozen booking seam — full
  03-0N plan files pending the blocker below). 4/5/6 not yet planned.
- **BLOCKED on Eli, two things (asked 2026-07-03):** (1) approval to run `pnpm install` + the two researched
  installs (posthog-js, @next/third-parties) so phase-2 execution can start — phase 2 needs NO design-reference
  and is otherwise buildable now; (2) push `design-reference/` from his MacBook (it was never committed — see
  03-STATUS.md) — phase 3 (the page) is fully blocked on it.
- **Model tiering NOW MANDATORY** post-incident: run execution agents on Sonnet, frontier for planning/checking
  (see [[../goals/momo-engine-hardening-2026-07-03]] incident 2026-07-03). Earlier today a Fable-5 limit killed
  both planners AND muted the bridge for 45 min; Eli switched the session to Opus 4.8.
- Execution model when unblocked: spawn a Sonnet agent per plan wave, each follows its PLAN.md, verify by the
  plan's command gates (build/lint:adherence/lint:css/vitest/playwright), commit locally, batch GitHub pushes
  for Eli's approval.
- Hard rules from the repo: every visual property from DS tokens (CI gates: no hex/px, no sizing @media);
  copy through `getLandingContent()` typed config; pinned stack versions in repo CLAUDE.md.

## Eli's standing answers (2026-07-03)
- **Pushes:** commit freely; EVERY GitHub push needs his OK first (batch at checkpoints). No standing
  exemption.
- **Inputs he owes, timed:** GTM/stape container IDs + tagging URL + cookie-domain (Fri AM);
  Supabase + Twilio credentials (Fri AM); AHPRA copy review ~15 min (Fri arvo/eve);
  Vercel + DNS session with him driving auth (Sat).
- AHPRA sign-off gates phase 3 exit and phase 6 exit — human review, non-negotiable.

## Deadline risks (from the repo's own STATE.md blockers)
- ACMA SMS Sender ID: NOT a v1 blocker if OTP sends from a plain/dedicated number (decide provider
  approach in phase 4; launch is Sunday — before the register matters for branded IDs).
- GTM/stape values gate phase 2 CLOSE (not its build — everything lands config-shaped with placeholders).
- Double-booking prevention + PHI-masked replay are phase 4/5 exit criteria — verification is in-flow.
