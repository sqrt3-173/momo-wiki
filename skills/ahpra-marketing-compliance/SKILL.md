---
name: ahpra-marketing-compliance
description: >
  Audits and rewrites marketing copy for AHPRA-regulated health services —
  clinic websites, landing pages, ads, EDMs, scripts — against the Health
  Practitioner Regulation National Law s133 advertising prohibitions, AHPRA's
  advertising guidelines, the Medical Board's cosmetic-surgery advertising
  guidelines, and adjacent ACL / super-release rules. Produces reworded,
  conversion-safe copy plus a human sign-off doc.
when_to_use: >
  Before publishing or editing any page, ad, landing page, EDM, or script for
  a health/medical/cosmetic/surgical client. Also use proactively during BD
  pipeline audits (auditing a prospect's existing site) and when drafting a
  "better build" to pitch. Applies to any regulated health service, but treat
  surgical/cosmetic/bariatric copy as highest-risk — it falls under the
  Medical Board's stricter 2023 cosmetic-surgery advertising guidelines.
---

## Why this exists

Both the clinic and the agency that wrote the copy are co-liable for a s133
breach — AHPRA doesn't stop at the practitioner. A breach can trigger
takedown notices, fines, and in serious/repeat cases registration action
against the supervising practitioner. This skill exists so MOMO catches these
before publish, not after a complaint.

## 1. The prohibited-pattern list

Each pattern below is a National Law s133 category or an adjacent regulator's
rule. For each: the plain-language rule, why it exists, and a grep hint to
find candidates fast (grep hints are a first pass only — every hit needs
human judgment, they will never catch everything and will over-flag).

### A. Testimonials about clinical care (s133(1)(c))
**Rule:** No patient stories, reviews, quotes, or "success stories" that speak
to clinical outcomes — including a clinic re-posting a patient's own review.
Comments about service/communication *without* a clinical reference (e.g.
"the front desk was lovely") are not testimonials and are fine.
**Why:** One patient's outcome doesn't predict another's; AHPRA treats these
as inherently misleading regardless of truthfulness.
**Grep hint:** `grep -riE "(changed my life|i lost [0-9]|my (surgery|procedure|journey)|thank you dr|★{3,}|verified patient|5 star)"` — also flag any embedded review-widget script/iframe and any first-person past-tense narrative block.

### B. Absolute/superlative claim words
**Rule:** No "safe", "painless", "guaranteed", "risk-free", "best", "#1",
"permanent(ly)", "proven", "miracle" used about the procedure or outcome
unless individually evidenced to the systematic-review standard AHPRA
applies (see Acceptable Evidence, below) — which a landing page essentially
never carries.
**Why:** These minimise real risk and create an unreasonable expectation of
benefit — the s133(1)(e) prohibition.
**Grep hint:** `grep -riE "\b(100% safe|pain(less|-free)|guarantee(d)?|risk-free|best|number one|#1|permanent(ly)?|miracle|proven results)\b"`

### C. Unqualified outcome or timeframe claims
**Rule:** No "lose Xkg", "healthy weight in a year", "reach your goal by
[date]" stated as a plain fact. Any outcome/timeframe claim must be (a)
evidenced, (b) paired with "Individual results vary", and (c) not presented
as typical/expected for everyone reading.
**Why:** This is AHPRA's own worked example of an "unreasonable expectation
of beneficial treatment or recovery" breach — exaggerating or giving
incomplete/biased outcome or timeframe information.
**Grep hint:** `grep -riE "\b(lose [0-9]+\s?(kg|kilograms|lbs)|healthy weight in|in (just )?[0-9]+ (days|weeks|months|year)|for good\b|keep it off)\b"`

### D. Before/after imagery
**Rule:** Never use without (i) separate, documented, revocable patient
consent specifically for advertising use (distinct from surgical consent),
(ii) identical conditions/lighting/pose and no editing, (iii) a "results may
vary for other patients" disclaimer on the image itself, (iv) a process for
the patient to withdraw consent and have the image pulled promptly.
**Why:** Medical Board cosmetic-surgery guidelines (2023) treat these as
high-risk for creating unrealistic expectations; TGA separately treats
before/afters implying a specific medicine/device outcome as a prohibited
representation.
**Grep hint:** not regexable — audit image filenames/alt text for "before"/
"after" and check the page for a consent trail and disclaimer text near each pair.

### E. Urgency / scarcity language
**Rule:** No "don't delay", "act now", "limited time", "spots filling fast",
"book before it's too late" — unless there is a genuine, clinically-driven
reason (there almost never is for elective bariatric surgery).
**Why:** s133(1)(f) — encourages indiscriminate/unnecessary use by manufacturing
urgency untethered to clinical indication.
**Grep hint:** `grep -riE "\b(don't delay|act now|limited time|hurry|spots (left|remaining)|before it's too late|last chance)\b"`

### F. Downplaying risk / omitting recovery reality
**Rule:** Any outcome or benefit claim needs risk and recovery context
reachable from the same page (doesn't have to be inline, but must be easy to
find) — this is non-negotiable for surgical procedures specifically.
**Why:** Medical Board cosmetic-surgery guidelines require "accurate,
realistic and educative information about risks" to be easily found; omitting
it is treated the same as an unreasonable-expectation breach.
**Grep hint:** no reliable regex — check whether the page (or one click away)
covers risks, complications, and recovery timeline.

### G. Inducements/gifts/discounts without stated terms
**Rule:** "Free consult", "$500 off", "bonus aftercare pack" is fine — but
the terms (eligibility, expiry, what's excluded) must be stated in plain
language, not buried. "Free" that's recouped elsewhere (price rise, Medicare
billing) isn't free.
**Why:** s133(1)(b).
**Grep hint:** `grep -riE "\b(free|% off|\$[0-9]+ off|discount|bonus|gift)\b"` then manually confirm adjacent T&Cs exist.

### H. Unsubstantiated comparative claims
**Rule:** "Most clinics do X, we do Y" needs on-file evidence (a real,
current, methodologically sound comparison) or it comes out. Vague
comparatives ("leading", "highest rated") without a named, checkable basis
are ACL misleading-conduct risk, separate from AHPRA.
**Why:** ACL s18/s29 — misleading or deceptive conduct, false comparative
claims about competitors' services.
**Grep hint:** `grep -riE "\b(most (clinics|patients|people)|better than|#1|leading|highest rated|unlike other)\b"`

### I. Superannuation / financial-outcome claims (bariatric-specific)
**Rule:** No copy that states or implies super will (or is likely to) cover
a procedure, or that early release is straightforward. This is a live,
named enforcement focus, not a theoretical risk.
**Why:** AHPRA and the ATO issued a **joint warning (16 Oct 2025)** targeting
exactly this pattern — clinics and marketers implying super access for
"overly expensive or unnecessary" procedures. ~30% of medical-category
compassionate-release applications are rejected by the ATO; framing access
as a given is both a s133 unreasonable-expectation issue and edges toward
unlicensed financial advice under the Corporations Act (ASIC territory).
**Grep hint:** `grep -riE "\b(your super|superannuation).{0,40}(cover|pay|fund|access)"`

## 2. Required-elements checklist

Run this against every page/asset before sign-off:

- [ ] Every outcome or benefit claim is qualified (not stated as a flat fact)
- [ ] "Individual results vary" (or equivalent) sits **next to** every outcome claim — not just in a footer
- [ ] Risk and recovery information is present or one click away
- [ ] "A GP referral may be required" (or equivalent eligibility language) appears near any booking/eligibility CTA for surgical/cosmetic services — reflects the Medical Board's mandatory independent GP-referral rule (effective 1 July 2023)
- [ ] Any pricing/financing figure (incl. super) carries its qualifying terms inline, not just linked
- [ ] Before/after images (if used) have consent trail, "results may vary" tag, unedited/matched conditions
- [ ] No first-person patient narrative or review content describing clinical outcomes
- [ ] No urgency/scarcity language without genuine clinical basis
- [ ] Comparative claims about competitors are either removed or backed by on-file substantiation
- [ ] **Realistic-expectations test**: read the claim as a worried, hopeful prospective patient — would a reasonable person walk away believing this outcome is typical/guaranteed for them specifically? If yes, it fails.

## 3. Review procedure

1. **Enumerate** every distinct claim on the page (headline, subheads,
   chips/badges, body copy, CTAs, image captions, alt text, structured data).
   Treat each as its own line item — don't batch "the hero" as one claim.
2. **Classify** each against Section 1's categories (A–I) plus plain
   false/misleading (s133(1)(a)) and unreasonable-expectation (s133(1)(e)).
   Tag: clear / borderline / breach.
3. **Resolve** each breach/borderline item one of three ways, in this order
   of preference:
   - **Wrap** — keep the claim, add the disclaimer/qualifier/T&Cs it's
     missing (fastest, works for most outcome-timeframe and pricing claims).
   - **Reword** — the claim itself has to change because no disclaimer fixes
     it (superlatives, permanence language, comparative claims without
     substantiation).
   - **Remove** — no compliant version serves the same purpose (testimonials,
     unsubstantiated urgency, before/afters without a consent process).
4. **Produce a sign-off doc** (one row per claim: original / rule risked /
   resolution / rewritten text / confidence) and route it to Eli for human
   sign-off before publish. AHPRA breach liability attaches to whoever
   controls the advertising — clinic and agency both — so this is never a
   silent auto-fix; a human confirms every resolution before it ships.
5. **Re-run after any copy edit**, not just at initial launch — this is a
   living check, not a one-time gate.

## 4. Sources

- Ahpra advertising hub / Testimonial tool — https://www.ahpra.gov.au/Resources/Advertising-hub/Resources-for-advertisers/Testimonial-tool.aspx (primary)
- Ahpra — Summary of the advertising requirements (primary)
- Ahpra — Guidelines for advertising a regulated health service (PDF, primary)
- Ahpra — Acceptable evidence in health advertising (primary)
- Medical Board of Australia — Guidelines for registered medical practitioners who advertise cosmetic surgery (primary, effective 1 July 2023)
- Medical Board — Cosmetic surgery FAQ: GP referral requirement (primary)
- Ahpra + ATO — Warning about extracting super early, 16 Oct 2025 (primary; directly relevant to bariatric marketing)
- TGA — Restricted and prohibited representations in advertising (primary; relevant if copy names a specific medicine/device, e.g. semaglutide, gastric band)
- ACCC — False or misleading claims (primary)

**Currency note:** several primary ahpra.gov.au / medicalboard.gov.au pages return 403 to automated fetch — cross-verified via search snippets + secondary commentary. Before relying on the exact wording of a disclaimer in a live campaign, view the primary PDF in a browser to confirm current phrasing. Researched 2026-07-04; re-verify the cosmetic-surgery guidelines and the super-release warning before each new campaign (fast-moving enforcement area).
