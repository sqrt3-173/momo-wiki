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

## Position (2026-07-04 ~07:30) — PHASE 3 SECTION BUILD UNDERWAY
- **Foundation DONE:** AHPRA compliance skill built ([[../skills/ahpra-marketing-compliance]] — caught the
  super-funding claim as a named AHPRA+ATO enforcement target); 6 flagged claims reworded (03-COPY-REWRITES.md,
  Eli-approved); DS atoms (Badge/Eyebrow) + enforced OutcomeClaim primitive ported; `getLandingContent()`
  extended with all 9 sections' compliant copy (03-01 content, 11 tests incl. a prohibited-phrase gate).
- **UI-SPEC signed off** (03-UI-SPEC.md). Eli's decisions: pricing $18,150/−12,000/−300 confirmed; real photos
  (in design-reference/uploads); **synced single booking form** (one shared state, rendered in both mid-page +
  bottom — his idea, better than my one-form+button); reword approved.
- **Sections built + wired + gate-green (tsc/adherence/css/build/fluid-sweep):** Hero (with AvCircle, BookCta,
  OutcomeClaim-wrapped headline, real surgeon/coach/patient photos, initials fallback for dietitian/nurse
  pending Eli's photo mapping) → booking `#book` anchor (form shell TODO) → Affordability. **Screenshot of the
  hero sent to Eli — looks professional.**
- **The proven section pattern** (all remaining sections follow it): Server Component `Section({content})`;
  copy from `page.<section>` props; DS tokens only (no raw hex/px/rgba — documented `eslint-disable
  no-restricted-syntax` for genuine sub-grid/hairline px, like the DS Button); `<a>`-free CTAs via the
  `BookCta` client leaf (scrolls to #book — `<button>` can't nest in `<a>`); OutcomeClaim wraps outcome claims;
  index-based list numbering (NOT a mutable counter — react-hooks/immutability lint bans it);
  `--text-on-dark` for muted white on the dark hero (no --border-on-dark token exists).
- **REMAINING (this is the ordered TODO):** sections Arc (§3.4 orbit diagram) · Surgeon (§3.5, real
  OliverFisherHeadShot.avif) · Pricing (§3.6 — INTERACTIVE toggles = client component + live-computed figures
  from base 18150/−12000/−300 ÷156wks) · Duration+Journey (§3.7 timeline) · Closing (§3.8, deep card) →
  assemble full page.tsx in v4 order → the booking form SHELL (BookingForm.tsx, 4 stages, `.ph-no-capture`
  wrapper, synced-form seam for phase 5) → check-compliance.sh gate + claim↔disclaimer parity + full fluid
  sweep → COPY-REVIEW.md for Eli's AHPRA sign-off (phase-3 exit gate). Then phases 4 (booking backend: Supabase
  + Twilio — needs Eli's creds), 5 (booking UI), 6 (deploy: Vercel+DNS session with Eli, Sat).
- **Playwright runs on port 3100** (`reuseExistingServer:false`) — a foreign bd-crm dev server owns :3000.

## Position (2026-07-04 ~05:30) — PHASE 2 DONE
- **Phase 2 (analytics + compliance): COMPLETE + verified** (momo_work phase 2 = verified). 4 plans built
  in the MAIN LOOP (subagents are guard-locked to read/web, so they can't execute code — execution is
  main-loop or, later, a governed path). Committed locally to the repo (NOT pushed — push needs Eli). Gates:
  31 vitest + 6 Playwright + adherence/css/build/full-lint all green. `pnpm`/pnpm@10.18.3 had to be installed
  on the mini (it wasn't present) + posthog-js + @next/third-parties (Eli-approved).
- **Playwright false-green LESSON:** `reuseExistingServer:true` on the default :3000 latched onto the
  long-running **bd-crm dev server** → tests ran against the WRONG app and passed falsely. Fixed: NV Playwright
  now runs on a dedicated **port 3100** with `reuseExistingServer:false`. Any project sharing a host must use
  a dedicated test port. (There is a stale bd-crm `next dev` on :3000 since 2026-06-23 — left running, not ours to kill.)
- **Phase 3 (landing page): in its UI PHASE.** Eli's instruction (2026-07-04): any phase with significant
  visuals goes through a UI-SPEC + his sign-off BEFORE building. A Sonnet subagent is drafting
  `.planning/phases/03-sydney-landing-page/03-UI-SPEC.md` from the v4 design; it goes to Eli for sign-off,
  THEN the visual sections (03-02/03-03) build. 03-01 (atom ports + OutcomeClaim primitive + content config)
  is toolkit/plumbing, buildable pre-sign-off.
- Dashboard: I update momo_work phase `stage` as phases progress (executing → verified) for Eli's live view.

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
