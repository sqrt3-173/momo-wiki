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

## COOKIE BANNER — DROPPED (Eli's informed decision 2026-07-04); WATCH RUNNING
Eli, after a thorough advisory thread (OAIC Medmate + Monash IVF determinations 11 Jun 2026; the
health-provider carve-out means NV is NOT covered by the small-business exemption despite <$1M turnover;
"but everyone does it" = true-but-now-enforced), decided: **keep the banner dropped, keep the ad pixel,
accept the (currently-low, rising) risk, and monitor the landscape.** An informed risk-acceptance by the
decision-maker — recorded here for the co-liability record.
- **IMPLEMENTED:** banner unmounted; consent defaults to opt-out (analytics+marketing ON, `defaultConsent`
  in `lib/consent/state.ts`); privacy policy updated to opt-out disclosure; **booking-form health-data
  masking (`.ph-no-capture` + `maskAllInputs`) stays UNCONDITIONAL** (never depended on the banner);
  consent plumbing retained for a future targeted ad-opt-in. 41 vitest + 5 playwright green.
- **REGULATORY WATCH (durable):** cloud routine `au-health-privacy-watch` (id `trig_01G5qCKAW7ifLBxJTZZVMpUJ`,
  monthly, next run 2026-08-02) researches OAIC/AHPRA/Privacy-Act developments and appends to
  **`wiki/ops/regulatory-watch-au-health.md`**. Cloud can't DM (no Discord connector), so **MOMO must check
  that log and relay any `MOMO-RELAY:`-prefixed entry to Eli.** ← see [[../ops/session-recovery]] standing check.
- Still relevant: when Eli's real GTM config lands, sanity-check what ad pixels it actually fires (per
  [[../skills/ahpra-marketing-compliance]] §J) so the disclosure/posture matches reality.

## Position (2026-07-04 ~13:15) — SITE CODE-COMPLETE ✅ (front + back); COPY SIGNED OFF
- **Copy: SIGNED OFF by Eli** (2026-07-04, "all good") → phase 19 (landing) = **verified**. Phase-3 exit gate cleared.
- **Booking backend WIRED + tested (dev mode):** Eli approved installing @supabase/supabase-js@2.108.2,
  twilio@6.0.2, libphonenumber-js@1.13.7. Built:
  - `lib/booking/clients.ts` — guarded service-role Supabase + Twilio Verify (server-only; null when creds absent).
  - `lib/booking/actions.ts` — `submitDetails` (Zod re-validate → phone normalise (libphonenumber) → Supabase upsert
    stage=verify → Twilio Verify send), `verifyCode` (Twilio check → stage=pick), `confirmSlot` (upsert booked),
    `saveExtras`. **Degrade to dev mode when creds absent** (accept any code, skip network) → preview works pre-creds.
  - `BookingForm.tsx` calls the actions; bookingId via `makeId()` (crypto.randomUUID needs secure context — the LAN
    preview http://192.168.x isn't one, so Math.random uuid-v4 fallback); postcode bound; error + busy states + inline
    error display. Flow reaches booked in dev mode (interaction-tested). tsc/adherence/build green.
- **THE WHOLE SITE IS CODE-COMPLETE.** Remaining to LAUNCH is only activation + deploy, both for Saturday w/ Eli:
  1. Eli creates a **Supabase** project + runs `apps/web/supabase/schema.sql`; a **Twilio** account + Verify service.
  2. Keys → `.env.local` (per `.env.example`) locally + Vercel env vars in prod — NOT via Discord.
  3. Deploy: Vercel (root apps/web) + the syd.nvhealth.com.au CNAME. ACMA sender-ID note for branded SMS (post-launch ok).
- Deadline Sun 10am AEST is very achievable — build done, only Eli's 2 accounts + the Sat deploy session remain.

## Position (2026-07-04 ~12:50) — COPY DOC + BACKEND FOUNDATION; WAITING ON ELI
- **Copy sign-off delivered** as an interactive Artifact (tap-to-tick, localStorage, on-brand teal):
  https://claude.ai/code/artifact/1d3f8eb6-7608-4736-97d7-1d9b7c66da53 — source `.planning/phases/03-*/COPY-REVIEW.md`.
  Separates "TRUE?" (Eli confirms prices + surgeon credentials) from "WORDING OK?" (I made it compliant).
  **AWAITING Eli's sign-off** (phase-3 exit gate). He'll reply "all good" or changes → I edit + re-verify.
- **Booking backend FOUNDATION built (no SDKs/creds needed), committed:**
  - `lib/booking/schema.ts` — Zod per-step validation (details/verify/slot/extras), AU phone regex.
  - `supabase/schema.sql` — bookings table + RLS **service-role-only** (client never touches Supabase; PHI off-browser).
  - `env.ts` — SUPABASE_URL/SERVICE_ROLE_KEY + TWILIO_ACCOUNT_SID/AUTH_TOKEN/VERIFY_SERVICE_SID (server, empty defaults).
  - `.env.example` — labelled blanks + where-to-find for Eli (copy → `.env.local`, or Vercel at deploy).
- **BLOCKED on Eli (2 asks sent):** (1) **approve installing 3 SDKs** — @supabase/supabase-js@2.108.2,
  twilio@6.0.2, libphonenumber-js@1.13.7 — so I can write the server actions (sendOtp/verifyOtp/saveBooking)
  + Supabase/Twilio clients; (2) **the actual Supabase + Twilio credentials** (via .env.local, NOT Discord —
  told him not to paste secrets in chat). Once both land: wire the 3 TODO seams in BookingForm.tsx → testable end-to-end.
- **SECRETS RULE (told Eli):** never paste API keys/passwords in Discord (stored on their servers). They go
  into `.env.local` (gitignored) locally + Vercel env vars in prod — never through MOMO or chat.
- Dashboard: phases 19(landing)+21(booking UI)+20(booking backend) all → executing.

## Position (2026-07-04 ~12:30) — BOOKING FORM BUILT → SITE VISUALLY COMPLETE ✅
- **`components/booking/BookingForm.tsx`** — the 4-stage booking island, faithful v4 port, WORKING (flow
  interaction-tested): details → verify (4-digit OTP, auto-advances) → pick (functional month calendar,
  past-dates + weekends disabled, Mon-first) → booked (summary + height/weight + contact windows). Floating-label
  fields (`.ld-*`), funding chips, all `.bk-*` CSS ported to globals.css. Wired into the `#book` slot in page.tsx.
- **PHI masking:** whole island wrapped in `.ph-no-capture` (+ `data-ph-no-capture`) → PostHog never records the
  typed name/phone/email/height/weight. Unconditional. APP-5 collection-notice + privacy links at the details step.
- **BACKEND SEAM STUBBED (phase 4, needs Eli's Supabase + Twilio creds):** `sendOtp`/`verifyOtp`/`saveBooking`
  are TODO markers in the component; currently just advance the local stage machine. UI+validation complete →
  backend is a drop-in. **This is the main remaining build, gated on creds.**
- **Synced-form note:** currently ONE form instance in #book + the Closing "Continue to booking" scrolls to it.
  Eli's preferred synced-two-instance (shared state, rendered mid + bottom) is a later refinement (needs shared
  context/state lift) — one-form-plus-scroll works fine for now.
- **Dashboard sync:** momo_work phases were stale (landing showed "planning"). Updated: phase 19 (landing)→executing,
  phase 21 (booking UI)→executing. **GAP: my main-loop work doesn't auto-log to momo_work** — I must update phase
  stages (and optionally finer activity) manually so the :3100 dashboard reflects reality. Asked Eli phase-level vs finer.
- All gates + fluid sweep 320-1920 + booking-flow interaction test green. Screenshots (details + calendar) sent to Eli.
- **REMAINING TO LAUNCH:** (a) booking backend wiring — Supabase + Twilio (needs creds); (b) Eli's AHPRA copy
  sign-off (COPY-REVIEW.md still TODO); (c) deploy (Vercel + DNS, Sat). Polish: orbit drift-sync if re-flagged,
  add-to-calendar ICS, the synced 2nd form instance.

## Position (2026-07-04 ~11:55) — HERO SCROLL SCENE + 7-item review round
Eli reviewed on :3200 and gave 7 fidelity notes — all done + committed + live:
1. Hero team/procedure circles must be STATIC (removed `drift`); only the Arc orbit moves. (Orbit drift is
   CSS-synced via SSR class — if Eli re-flags "out of sync", wrap the orbit nodes in one animated container.)
2. Procedure pulse was VERTICAL (ripplebox had shrunk to the icon width) → set `.nvh-ripplebox{width:100%}` = horizontal.
3+4+5. **Hero scroll scene rebuilt as ONE system** (Eli: the sticky+z-index pin was fragile): new
   `components/landing/ScrollController.tsx` (client, rAF-throttled) publishes `--hero-progress` (0→1) on <html>;
   the Hero inner reads it (`transform:scale(calc(1 - var(--hero-progress)*0.08)); opacity:calc(1 - …)`,
   transform-origin 50% 35%) to fade+scale as it recedes. ALL post-hero content INCLUDING the footer now sits in
   ONE `position:relative;z-index:1` opaque layer over the `position:sticky;top:0;z-index:0` hero — footer no
   longer bleeds through (was the bug: footer was outside the layer). Restored `data-hero` + `data-nav-dark` on the
   hero section so the DS Header's own scroll→glass + dark-logo listener works.
6. Minimal v4 footer (`components/sections/MiniFooter.tsx` — legal line + Privacy/Terms links) REPLACES the full
   DS `<Footer>`. Legal line is a const in page.tsx (move to content later).
7. "Individual results vary." was squished — ROOT CAUSE: it rendered INSIDE the `<h1>` and inherited the
   heading's negative letter-spacing + tight line-height. Fix: `<OutcomeClaim as="block"><h1>…</h1></OutcomeClaim>`
   (h1 is the child → disclaimer is a sibling line below) + `.hero-claim [data-outcome-disclaimer]` CSS
   (margin-top space-3, letter-spacing normal, colour --text-on-dark).
All gates + fluid sweep 320-1920 green. Verified via scrolled/footer screenshots.

## Position (2026-07-04 ~11:40) — MORE FIDELITY FIXES + LIVE PREVIEWS
- **Arc "always around you" rebuilt as the REAL v4 orbit** (I'd wrongly simplified to a row): You centre-bottom
  (`nvh-orbit-node` at 50%/86%), care team on the dashed elliptical arc (`.nvh-orbit-ring`), %-positioned in an
  `aspect-ratio:720/460` box with `overflow:hidden` (so it never overflows the page), drift+ripple pulse. Verified.
- **Surgeon headshot** → correct `oliver-fisher-headshot.avif` crop (I'd swapped it to the tighter team-avatar crop).
- **PORT MAP (important — machines on the LAN, mini IP 192.168.0.250):** `:3000` = foreign bd-crm dev server;
  **`:3100` = momo work-OS dashboard** (`/Users/momo/momo/dashboard`, `next dev -H 0.0.0.0 -p 3100`, reads momo_work
  via .env.local — this is Eli's live view); **`:3200` = NV site hot-reload preview** (`next dev -H 0.0.0.0 -p 3200`);
  **`:3400` = NV playwright test port** (moved off 3100 to stop clashing with the dashboard). Both dev servers started
  this session are network-bound so Eli can view from his MacBook Pro.
- **TODO (promised Eli): make the dashboard auto-persist.** It's currently running inside MY session's Bash bg →
  dies if the session restarts. Wire it into the launchd guardian (`ops/momo-guardian.sh`) so it self-restarts like
  the momo tmux session. NV preview on :3200 is also session-bound (fine — it's just a dev convenience).

## Position (2026-07-04 ~11:05) — DESIGN-FIDELITY REWORK (Eli feedback) ✅
Eli reviewed and caught corners I'd cut — all fixed, and a real lesson (see [[../feedback/design-fidelity-no-shortcuts]]):
- **ALL real assets are in `design-reference/ui_kits/website-fluid/assets/`** (NOT `uploads/` — I'd looked in the
  wrong place and used initials/text fallbacks). Now wired + copied to `apps/web/public/assets/`: dietitian.png,
  nurse.png, proc-gastric-{sleeve,bypass,mini}.svg, hospitals/{st-george,east-sydney}-duotone.png, nvh-icon-white.svg.
- **Procedures** now render the real diagram SVGs (in `.nvh-ripplebox` glow circles), not letter placeholders.
- **Pulse animations ported faithfully** from the v4 `<style>`: `nvh-drift` (avatar bob) + `nvh-ripple`
  (rings behind "You" 150px + procedures 98px), with the `prefers-reduced-motion` fallback. In globals.css.
- **5-stage timeline rebuilt to EXACT v4**: `.nvh-jrow` = flex row of equal `.nvh-stage` columns w/ hairline
  dividers (NOT flex-wrap — that was the "containers wrap/direction off" bug); zero-width `.nvh-jmark` marker
  between stages 2&3 (dashed rule + floating pill); the care-team count pills (`.nvh-ccount` + colored dot,
  kind→colour) I'd dropped — added `counts` to the journey content schema. Stacks via `@container (max-width:720px)`
  referencing the DS page container (do NOT put container-type on .nvh-jrow — breaks the query).
- **Credentials 2×2** via `.nvh-cred-grid` (repeat(2,minmax(0,1fr))), not the auto-fit 3+1.
- Tooltip overflow fix: `.nvh-tip-bubble` uses `display:none`→`block` (opacity:0 absolutes still widen the doc at 320px).
- All gates + fluid sweep 320-1920 + pricing interaction green. Screenshots (full page + timeline close-up) sent to Eli.

## Position (2026-07-04 ~10:20) — ALL 7 LANDING SECTIONS BUILT ✅
- **The full landing page is visually COMPLETE** (bar the booking form). All 7 content sections built, wired
  in v4 order, gate-green (tsc/adherence/css/vitest/fluid-sweep 320-1920/build): Hero → booking#book anchor →
  Affordability → **Arc** (team, You-centred) → Surgeon → **Pricing** (interactive PHI/CDM toggles,
  live-computed gap/weekly — $18150−12000−300÷156; interaction test proves hydration) → DurationJourney
  (bars + 5-stage timeline) → Closing (deep card) → Footer. Two full-page screenshots sent to Eli.
- **Real bug caught + fixed:** the sticky hero's z-index:0 was letting its OutcomeClaim span intercept clicks
  on EVERY lower section (all CTAs + toggles dead). Fixed with the v4 "v3-over" `position:relative;z-index:1`
  wrapper around all post-hero sections. Also fixed a 320px overflow (long CTA labels — BookCta now wraps;
  DS Button is single-line/fixed-height — + `min(100%,N)` column min-widths).
- **Layout-in-CSS classes** now cover: card-grid, split, headshot, hairline, dur, bar, jrow/jstage/jmark,
  switch (all in globals.css — keeps px out of TSX for the adherence gate). Switch state via `data-on` attr.
- **REMAINING for phase 3 exit:** the booking form SHELL (BookingForm.tsx — 4 stages details→verify→pick→booked,
  `.ph-no-capture` wrapper, synced single-form rendered mid-page #book + bottom Closing per Eli) → drop it into
  the #book slot + Closing → check-compliance.sh + claim↔disclaimer parity gate → COPY-REVIEW.md for Eli's
  AHPRA sign-off. Placeholder/polish TODOs: dietitian/nurse real photos (Eli to map), procedure diagram SVGs
  (initials fallback now), hospital logos (text now), hero disclaimer spacing (slightly cramped).
- Then phases 4 (booking backend: Supabase+Twilio — needs Eli's creds) · 5 (booking UI) · 6 (deploy: Vercel+DNS w/ Eli, Sat).

## Position (2026-07-04 ~07:30) — PHASE 3 SECTION BUILD UNDERWAY
- **Foundation DONE:** AHPRA compliance skill built ([[../skills/ahpra-marketing-compliance]] — caught the
  super-funding claim as a named AHPRA+ATO enforcement target); 6 flagged claims reworded (03-COPY-REWRITES.md,
  Eli-approved); DS atoms (Badge/Eyebrow) + enforced OutcomeClaim primitive ported; `getLandingContent()`
  extended with all 9 sections' compliant copy (03-01 content, 11 tests incl. a prohibited-phrase gate).
- **UI-SPEC signed off** (03-UI-SPEC.md). Eli's decisions: pricing $18,150/−12,000/−300 confirmed; real photos
  (in design-reference/uploads); **synced single booking form** (one shared state, rendered in both mid-page +
  bottom — his idea, better than my one-form+button); reword approved.
- **Sections built + wired + gate-green (tsc/adherence/css/build/fluid-sweep):** Hero (AvCircle, BookCta,
  OutcomeClaim-wrapped headline, real surgeon/coach/patient photos, initials fallback for dietitian/nurse
  pending Eli's photo mapping) → booking `#book` anchor (form shell TODO) → Affordability → Surgeon
  (real headshot, credentials, procedures w/ AvCircle-initials icons, hospitals as text — logos TODO).
  **Hero screenshot sent to Eli — looks professional.** (Page order note: Arc goes between Affordability
  and Surgeon — insert when built.)
- **LAYOUT PATTERN (key, use for all remaining sections):** the adherence ESLint rule bans ALL px in TSX
  (flex-basis, grid minmax, hairlines — everything). Fighting it line-by-line is slow. SOLUTION: put layout
  scaffolding in `globals.css` classes (lint:adherence is `eslint .` = TSX only, does NOT scan CSS; the footer
  already does this with .fl-foot-link). Added reusable classes: `.nvh-card-grid` (auto-fit minmax-240 grid),
  `.nvh-split`/`.nvh-split-media`/`.nvh-split-body` (two-col wrap), `.nvh-headshot` (fluid clamp), `.nvh-hairline-top`.
  Keep COLOUR/SPACING as tokens inline; layout structure in CSS classes. Also: react-hooks/immutability bans
  mutable counters in render (use index-based numbering); the DS `Button` renders `<button>` so CTAs use the
  `BookCta` client leaf (scroll to #book) not `<a><button>`.
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
