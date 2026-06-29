# Bariatric BD Intelligence

Exhaustive intelligence asset over the Australian bariatric surgeon market, built on the
BD-CRM Postgres graph. Source: Eli's two Monday.com exports (220 surgeons, 118 hospitals).
Goal: enrich every surgeon across ~15 dimensions + map the relationship graph + spot
NV Health follow-up-program openings (clinics that removed/lack dietitians).

Project repo: `/Users/momo/momo/projects/bd-crm`. See also [[bd-crm]].

## Status (2026-06-24) — COMPLETE
**All 218 surgeons enriched at DEEP depth** (the 219th DB row, "New Group", is a junk seed
to delete). Full-market run done unattended over one extended session, region by region
(SEQ → Greater Sydney → Greater Melbourne → Greater Perth → Adelaide → Regional NSW →
Hunter → Illawarra → Hobart → Darwin → ACT → Regional QLD).

Final graph: 218 surgeons · 299 clinics · 312 hospitals · 465 named allied-health staff ·
110 allied-health companies · 99 marketing agencies · 136 referrers · **2,150 edges** ·
451 social profiles · 425 reviews · **569 price points (91 surgeons publish pricing)** ·
148 Wayback findings.

**Market-wide reads:** (1) reputation capture is the near-universal #1 gap — even
high-volume surgeons sit on near-zero managed Google reviews; (2) web/dev market is
unconsolidated — 99 agencies, long tail of one-off shops, only **Your Practice Online
(11 sites, templated/dated)** is a real incumbent = prime displacement pool; (3)
"rented vs owned MDT" cleanly separates strong clinics (named in-house team: Morphē,
Adelaide Bariatric Centre, Whiting/Townsville) from the majority who rent dietetics/psych.

### Earlier milestone (2026-06-23)
Pilot 20 complete at DEEP depth: 20 surgeons, 11 labels corrected, 18 published prices,
23 Fresh Start edges, 135 research edges. (Superseded by the full run above.)

Eli's feedback incorporated: deeper enrichment (named individuals + profile URLs, financing,
accreditations, named referrers/competitors, ranked BD angles); label-overwrite allowed;
entity profile pages + cross-links; **Obsidian-style node graph** at `/graph` with
geographic layout (north=up: Brisbane↑Sydney↑Melbourne, Perth far left), hover edge
explanations, click-to-pin + connection list.

**Scaling reality:** each deep agent ≈ 40k tokens through the main-loop context (results must
pass through context to be persisted — subagents can't write files/DB). 20 surgeons is a heavy
single session; the remaining ~200 is a long multi-session/background job, not a one-sitting run.

## Architecture
- Graph schema in `prisma/schema.prisma`: Surgeon, Clinic, Hospital, AlliedHealthStaff,
  AlliedHealthCompany, MarketingAgency, Referrer + generic `Edge` (typed relationships) +
  enrichment tables (WebPresence, AdPresence, SocialProfile, Review, PricePoint,
  WaybackFinding). Every fact carries `Confidence` (CONFIRMED/LIKELY/INFERRED/UNKNOWN) +
  source + asOf. Eli's board commentary preserved as seed truth, never overwritten.
- Surgeons UI: `app/surgeons/` (list w/ filters: region, Fresh Start, ads, NV Health
  opening; detail w/ full enrichment + connection graph). Markdown report: `enrich/report.ts`.

## Enrichment pipeline (the reusable method)
Hybrid, because background subagents are sandboxed to web tools only (no Bash):
1. **Research agents** (parallel, WebSearch+WebFetch) per surgeon → strict JSON: site
   discovery, reviews, social cadence, allied-health + INFERRED employment, agency,
   referrers, connections, pricing. Honest-confidence mandate (no guesses as fact).
2. **Deterministic scrapers** (`enrich/techstack.py`, `enrich/wayback.py`) run by the main
   loop → tech stack, **ad pixels (reliable free ads signal)**, socials, team links,
   Wayback removed-dietitian detection. `enrich/augment.py` merges scrapers into agent JSON.
3. `enrich/write.ts` persists to the graph (resolves/creates edge targets by name).

## Key learnings
- **Scaling constraint:** every agent result passes through the main-loop context to be
  persisted; ~12 surgeons ≈ heavy. All 220 can't run in one session — full send needs
  batching across sessions or a background Workflow (transcripts kept out of context).
  Allowlisted WebSearch+WebFetch for subagents (Bash stays blocked by the auto-classifier).
- **Pilot caught errors in Eli's board** (Gunawardene not WLSA/Fresh Start → uses Liora
  Health; Jameson does have a clinic) — the enrichment is a data-quality check too.
- **Free tier gets:** tech, ads, team/allied-health, Wayback, published-or-not pricing,
  socials, agency credits. **Needs paid later:** Google star ratings, Google Ads intel.
- Fresh Start = external Kajabi-delivered 2-yr program; partners dated (Russell 2014,
  Taylor 2018, Lewis 2021). Other programs found: Liora Health, CompleteHealth/HWA.

## Cleanup DONE (Eli's calls actioned 2026-06-24, `enrich/curate.ts`)
Result: **216 surgeons, all enriched.** Region recuration (11→Greater Sydney Area, 5→Hunter,
Cosman→Illawarra, Ho→Regional NSW, Clough→Greater Melbourne; Fazal already QLD/SEQ; Abdullah
Shepparton + Murray Launceston left in nearest bucket — no regional-VIC/north-TAS bucket
exists). Ahmad Aly Dr/Mr merged (kept Mr, +17 edges; Dr deleted). Wendy Brown, Frydenberg,
Jon Armstrong → clinicStatus INACTIVE (prior status preserved in correctionLog). "New Group"
junk deleted. Christopher Lim deleted (HPB/cancer, not bariatric). Patiniotis kept + flagged
in correctionLog (A$755k negligence judgment). Still open: AlliedHealthCompany dedupe
("Fresh Start" vs "The Fresh Start Program"); clinic-name-as-company leak.

## Historical pricing workstream (2026-06-24) — done, with a hard external limit
Eli: gather historical pricing (price-over-time + forum/review mentions). Stored in PricePoint
with `note LIKE 'HISTORICAL%'`, asOf = the historical date, context tag [WAYBACK|FORUM|REVIEW|OTHER].
Scripts: `enrich/hist_price_schema.md` (agent brief), `enrich/histprice_wayback.py` (deterministic
CDX+snapshot scraper), `enrich/write_histprice.ts` (persist, idempotent per-surgeon).

**KEY LEARNING — archive.org access asymmetry + rate limit.** The research subagents' WebFetch
is HARD-BLOCKED by web.archive.org (every snapshot-content fetch fails; they can only read the
CDX *availability* API = snapshot dates, not bodies). The main-loop python CAN read snapshot
bodies via the raw `…/web/<ts>id_/<url>` form — BUT only for sporadic requests; under sustained
bulk scraping archive.org throttles content fetches (~8s/query, then intermittent failures), so a
clean 77-domain sweep isn't achievable in one pass. Net: deterministic main-loop scraper (regex
only, no injection surface) is the right engine; agents are useless for archive content.

**Findings (real dated figures recovered where archives were readable):**
- **Centre for Weight Loss (Dhir/Foo):** insured OOP FLAT 2019→2021 (sleeve $6,800, bypass $8,000,
  band $5,800); by 2026 lap-band dropped, sleeve→~$7,500, bypass→~$8,500, uninsured now published
  ($19,900 sleeve). The cleanest band-era→sleeve-era trajectory.
- **Lap Surgery Australia (Chong/Saravanamuttu/Cheah):** recent CUT — insured sleeve $7.5–8k→$7,200,
  self-pay $20.5–22.5k→$19,600; Allurion added.
- **Brisbane GO Surgery (Jason Wong):** consult $250 stable 2022→2023, components shift to 2025.
- **Structural headline:** most AU clinics didn't publish prices online pre-~2023 (ABC, Upper GI
  cost pages first archived 2023); pre-2023 was "call for a quote." Forums had ~no dated AU $ figures.
- **BD read:** pricing flat-to-slightly-DOWN, not up. Real movement is structural — lap-band pricing
  vanishing, uninsured/self-pay going public, balloons added (competition + GLP-1 pressure).

DM'd Eli the summary 2026-06-24. Coverage: 6 surgeons with dated historical rows (60 rows). Can
deep-dive any single clinic's full archive on request (politely, to avoid the throttle).

## (superseded) Cleanup queue — awaiting Eli's curation call, DM'd 2026-06-24
- **Region buckets messy:** ~10 "Regional NSW" seeds are actually metro Sydney (Devadas,
  Krishna, Durmush, Sulman Ahmed, Fedorine, Kuzinkovas, Khalid Ahmed, Siddaihah); Shirhan
  Ho → Bowral; David Murray → Launceston (not Hobart); Anthony Clough → Melbourne (not
  Perth); Waqas Fazal → QLD (not Melbourne); Abdullah → Shepparton/regional VIC. Agents
  corrected primaryCity/State with provenance but cannot touch greaterRegion — needs a
  taxonomy recuration pass.
- **Duplicate:** Ahmad Aly appears twice (Dr + Mr rows). Also earlier: duplicate "Dr Craig
  Taylor" from the source board.
- **Retired from bariatrics:** Wendy Brown, Harry Frydenberg, Jon Armstrong (handover to
  Sundararajan 19 Jun 2026).
- **Junk row:** "New Group" (empty seed) → delete.
- **Fit/risk flags:** Christopher Lim (Canberra) is really HPB/cancer, weak bariatric fit;
  Patiniotis (Hobart) carries a A$755k malpractice judgment dominating his SERP.
- **AlliedHealthCompany dedupe:** "Fresh Start" vs "The Fresh Start Program"; a clinic name
  leaked in as a company.

## Entity deep-enrichment (2026-06-25) — non-surgeon entities as their own profiles
Eli: enrich agencies / allied-health cos+staff / financing systems "as in-depth as possible."
Pipeline (reusable): `enrich/entity_schema.md` (agent brief) → locked web+read agents → strict
JSON → `enrich/harvest_entity.py` (format-agnostic transcript walker — the surgeon-era
extract_agent/harvest.py do NOT parse these newer .output files) → `enrich/write_entity.ts`
(dossier→entity.notes prefixed `ENRICHED <date>`; socials/reviews/edges to the generic
entityType-keyed tables; edge method='entity-enrich'). Value-first waves, ≤6 concurrency,
~12-fetch cap, ScheduleWakeup backstop per wave.

**Enriched (high-value tiers):** 24/99 marketing agencies (all reach≥2), 6/110 allied-health
companies, 5/136 referrers/financing, 7/465 allied-health staff (all the multi-surgeon shared
ones). ~41 new entity→entity/surgeon edges. **Long tail (~100 single-clinic companies + ~400
single-mention sole-trader staff) SAMPLED, not exhaustively enriched** (low ROI/record).

**Findings:**
- **Agencies (3 tiers + lock-in):** dominant incumbent = **Your Practice Online = Quantum
  Digital** (same parent, Mark Ryan/Dr Prem Lobo; ~15 surgeons on one templated offshore
  Bangalore-built product). Medical-niche specialists: Practice Boost, Online Medical
  (surgeon-founded), Medsolve, Digital Practice, Splice (50+ clinics, marquee Cochlear/RACGP/AMA),
  WebInjection (boutique, 15-mo waitlist). Rest generalists. Real platform lock-in only at
  **OracleStudio (proprietary Xargo CMS, sites on *.xargocms.com)** + **TEAPOT (proprietary CMS)**.
  Everyone "custom"-markets but builds template WP/Webflow/Squarespace — lock-in is retainer+hosting,
  not build. Clients churn between agencies. MOST = DISPLACE; PARTNER candidates = Concept Marketing
  + Splice. Localsearch = just a directory, not a real competitor.
- **Fresh Start = the standout:** 22 surgeons rent the same Kajabi-delivered aftercare program
  (Amber Kay coaching team), per-patient license bundled as "free." No owned moat → the single
  best owned-aftercare-platform displacement for NV/clinic-OS.
- **Financing = channel:** SuperCare (dominant early-super-release, thin moat — could in-house);
  TLC/MacCredit (Tim Boon, same founder) loan rails; Medtronic 'Science of Obesity' = device-vendor
  funnel (being decommissioned), useful only as a free AU practice roster.
- **Rented-vs-owned MDT is the spine:** dietetics/psych COMPANIES (360-me/CSIRO, Weight Management
  Psychology/Glenn Mackintosh, Healthy Lifestyles, Psychological Services Group) = PARTNER/white-label
  (value = named credentialed clinicians, hard to displace). The shared DIETITIANS (Ashleigh Gale ×8,
  Kathy Benn ×6, Timmy Chu ×5-6, Penny Weigand ×4, Cassie Stuchbery) are each one contractor wired
  into many competing surgeons = the rented MDT embodied → win the dietitian = warm access to all
  their surgeons. **Adelaide Bariatric Centre** = rare OWNED in-house MDT (the benchmark).

DM'd Eli: agency-class summary + companies/financing/staff summary + (on request) a YPO/Quantum
Digital surgeon ranking by reputation with NV Health partnership flags (top: David Martin —
elite surgeon, invisible online; Hatzifotis/Huo/Taylor — top surgeons on a rented template).

## Allied-health EXHAUSTIVE enrichment + per-dietitian fees (2026-06-28) — COMPLETE
Eli chose the exhaustive option specifically to learn **"how much each dietitian charges"** as
structured, queryable data. Ground-out the entire long tail: **all 465 AlliedHealthStaff + all 110
AlliedHealthCompany now ENRICHED (0 remaining in either table)**, captured per-practitioner
CONSULTATION FEES as `PricePoint` rows (note prefix `FEE`, isPublished flag, Medicare rebate in note).
Method: same locked web+read agent pipeline (`entity_schema.md` → `harvest_entity.py` →
`write_entity.ts`), ≤6/wave (8 on final waves), 2–5 fetch cap (down from ~14 — the lower cap killed
the 600s-watchdog stalls), completion-driven (bank+launch next wave on return; ScheduleWakeup only a
failsafe). ~60 staff waves + 17 company waves over one continuous session.

### THE ANSWER — dietitian initial-consult published fees (~46 practitioners who publish)
Range **$95 → $295**, median **~$185–190**, bulk cluster **$150–210**. Reviews/follow-ups ~$70–140.
- Cheapest: **Carolyn Shoemark $95**; Stevie Raymond (telehealth flat) $115; Storm Law $130 (inferred);
  Peita Hynes / Annalie Houston $145; Gemma Campbell / Mary-Anne Chamoun / Helen Bauzon / NQOSC / Mari
  Harrison $150.
- Mid: Loretta Bufalino $165; Cassandra Robins(Ipswich) / Sophie Sellars / Zoe Cooke / Carly Barlow $170–180;
  Rhiannon Dick $175; **360 Surgery (Teagan Hollis/Jeanne Fourie) + Heidelberg (Noela Kluchkovsky) + WMP
  (Shauna Summers) + Kerryn Chisholm $180**; Sally Lane $185; **Morphē team (Sally Johnston, Jennifer,
  Justine, Kahlia) + NQMIS team (Melissa Whiting, Allison Hillery, Jess Proudford) + Claudia Jahjah $190**;
  Canberra Allied Health $195; Happy Apple $197; Amanda Clark $199; Suraya Nikwan $200.
- Top: **Balance Nutrition tiered — Dietitian $210 / Senior $245 / Director (Leah Stjernqvist) $295** (the max).

**KEY STRUCTURAL FINDING:** most allied-health DON'T publish standalone fees — dietetics/psych are
**bundled into the surgery package or phone-quoted**. Only independent dietitians, some psychologists,
and a few bariatric GPs publish. Every published fee is now a queryable PricePoint row (sort/compare anytime).
Context bands: psychologists $130–320 (bariatric pre-op assessments $240–280; e.g. Veritas/Sebastian Lean
Jerald $250, Be Happy $250); bariatric GPs bulk-billed/free→$400 (Crane $400, Atkinson $385); endocrinologist
$355–465 (Chesterman); exercise phys $95–195; NP $135 or free. Surgery: insured sleeve OOP ~$2.5–8.8k,
self-funded sleeve ~$15.5–30k; Allurion balloon program $5–8.2k. Aftercare: Fresh Start $275/$475/$675 (2yr),
Veritas Silver $1000/Gold $2000, Adelaide Bariatric $3,800 (2yr).

### BD theses (companies phase — the strategic layer)
- **Financing channel (bolt onto any build):** **SuperCare** = dominant early-super-release facilitator
  (~$480 flat, no-approval-no-fee, partners 12+ bariatric clinics) + **TLC / Total Lifestyle Credit** =
  patient loan rail (6.99% from, $2k–70k). Both reusable on any Eli clinic site → lift conversion of
  cash-short patients at ~zero cost to the clinic.
- **Outsourced aftercare/clinic-OS layers to DISPLACE/internalise:** **Fresh Start Program** (Kajabi,
  licensed to 19 clinics, $275–675/patient/2yr — quantifies the recurring spend a clinic could internalise);
  **My Weight Doctor / Alevia** (national *licensable clinic-OS SaaS network* — e.g. Orange Healthy Weight
  Clinic is a licensee renting its whole portal/dietetics/programs); **NV Health** (productises the 24-mo
  allied-health layer at **$1,950–2,750/patient** — direct pricing benchmark for a clinic-OS); **NFWLS**
  (Sally Johnston's per-patient B2B nutrition content, Elevate $497); evolvme (Adelaide aftercare practice).
- **Web-rebuild pipeline:** pervasive rented/template/broken web — PatientUplift, HomeGiraffe, Your Practice
  Online, Smart Space, OracleStudio, Elevate, Converge, Lasso Creative, Blue Road Group, Squarespace, Wix,
  Joomla, SEO-doorway templates, stale/duplicate domains, and **a broken TLS cert (Bariatric Medicine
  Integrated)** + exposed SiteGround staging clone (The Specialist Haus). Easy "draft something better" entry.
- **Rented-vs-owned MDT (confirmed at scale):** shared dietitians wired into many competing surgeons
  (Ashleigh Gale, Kathy Benn, Tania Chaanine, Penny Weigand, Vanessa Jenkin; psychologists Warren Artz, Cal
  Paterson, Glenn Mackintosh) = win the practitioner → warm access to all their surgeons. Vertically-integrated
  clinics that OWN their MDT (Morphē, Adelaide Bariatric Centre, NQMIS/Whiting, Aurora, Veritas, Upper GI
  Surgery, OClinic) = benchmark/compete, not displace.
- **SRC accreditation** (Center/Surgeon of Excellence) = prospect-qualification signal — flags serious,
  higher-volume, marketing-conscious clinics worth prioritising.

### Data-hygiene notes for next pass
- Near-dup nodes were quick-tagged + merge-flagged in their dossiers (act on canonical): "Fresh Start
  Program" vs "The Fresh Start Program"; "Total Lifestyle Credit (TLC)" vs "TLC Medical Finance"; SuperCare
  vs "SuperCare (MySuperCare)"; "Orange Healthy Weight Clinic" vs "(OHWC)"; "Metabolic GI Surgery Brisbane"
  vs "(formerly Obesity Surgery Brisbane)"; "Darebin Weight Loss Surgery" vs "...MDT"; two "Shelley Lisle"
  rows; placeholder nodes ("Multidisciplinary team (unnamed)", "Unnamed in-house dietitian", St Vincent's
  Northside pool). "Sally" (Morphē) ≈ "Sally Johnston" likely same person. Misclassified-as-surgeon staff
  nodes flagged (Aaron Lim, Denbigh Simond, etc.). Tangram Health misclassified (it's a musculoskeletal
  physio group, not bariatric). The Toonson "Sally Johnston" link was false (his site only cites her book).
- DM'd Eli the final report 2026-06-28 (led with the dietitian fee answer); offered to rank top
  displacement/partner targets next.

## St George Private — bariatric-surgeon politics map (2026-06-28)
Eli asked whether there are politics among the St George Private bariatric surgeons, then for a
"ballistic depth" public-signal dive on each (socials, research, events, sentiment, medico-legal,
personality/motive) to infer the political angles. 15 surgeons profiled via 1 research agent each
(web+read, 8-fetch cap). **No documented feuds exist in public signal** — every camp/tension below
is STRUCTURAL INFERENCE from titles/credentials/brand positioning, not observed conflict. The value
is the *shape* of the board, which is a textbook politics setup.

### Three power crowns (one now vacant)
- **A/Prof Michael Talbot — governance king.** Chairs Dept of Surgery + Medical Advisory Committee
  at the Private (credentials everyone in the building) + Head of Upper GI at St George Public +
  A/Prof UNSW + ANZMOSS Past Pres + SRC Master Surgeon (2012). Power = institutional control +
  academic eminence. Most powerful figure.
- **Dr John Jorgensen — directorship king.** Hospital-wide title "Director of Bariatric Surgical
  Services" + highest-volume narrative + own consumer brand; no academic title. Power = admin +
  volume. **Central tension = two co-equal founders, two crowns (governance vs directorship).**
- **Dr Ken Loi — late statesman; DIED Oct 2023.** Was ANZMOSS treasurer / IFSO-APC pres-elect /
  RACS-NSW chair + "St George Obesity Surgery" brand. His death = succession vacuum = likely the
  real engine of any recent "politics."

### Camps
- **Talbot academic bloc ("Upper GI Surgery")** — dominant; owns research, COE branding, society
  offices, governance. Members: Talbot (lead), Jason Maani (SRC Master, protégé/heir, building own
  SEO brand = mild internal tension), Daniel Chan (academic engine — PhD candidate, h-index 8,
  military/Order of St John, multilingual Chinese-Australian niche), Adam Yee (loyal senior assoc),
  Georgia Rigas (obesity physician, not surgeon).
- **Jorgensen** — own practice + directorship sitting above all practices; bloc of one.
- **Succession claimants & commercial independents (the "money" cohort — fight for patients, not
  governance):** Qiuye Cheng (inherited Loi's brand; asset is Loi's *name* not own eminence; not on
  hospital SRC roster), **Mark Magdy (BEST BD beachhead** — most aggressive marketer: TikTok/Linktree/
  Allurion, solo, no ANZMOSS/research, weak infra vs ambition), James Chau (4,000+ cases, genuine SRC
  Master but under-markets, multi-site regional; revenue competitor not political rival), Roy Hopkins
  (young, oncology-credentialled, flanks regional Nowra/Wollongong, thin digital, accommodates not
  fights), Amitabha Das (dual-brand serious-HPB + commercial bariatric funnel; real base SW Sydney /
  HoD Bankstown; uses St George halo only).
- **Maverick founder:** Aravind Kuzinkovas (Advanced Surgicare family business, independent).
- **Above the turf war:** David Yeo (cancer/transplant academic, bariatrics a sideline),
  Oliver Fisher (early-career academic). Low bariatric stakes.
- **Peripheral / not a player:** Fedorine (SW Sydney generalist, legacy St George privilege only).

### Verdict on Eli's hypothesis (Yeo fine / Fisher high-ethics-stays-out / "driven by money, others money-hungry")
- Yeo above-it: **strongly correct** (sideline stake). Fisher: **supported w/ asterisk** (real outlier
  but quietness partly = early-career; "high ethics" unverifiable, partly self-positioning).
- "Money-driven": **half right.** Money game is real for the independent cohort (his instinct nails
  them). But the *deepest* power axis is control/prestige (Talbot's governance grip + post-Loi
  vacuum), not fees. Two tracks: founders fight for control+reputation; independents fight for patients.

### Corrections logged
- Ken Loi deceased (Oct 2023). Maani firmly Talbot-camp (earlier "straddles two camps" was wrong).
- "Hopkins = ANZMOSS past-president" claim online is FALSE (search-engine conflation with Talbot).
- Chau's 2013 J.Obesity paper = a different (Edmonton) Chau. Magdy NOT a Fresh Start partner (that's
  the Martins). ~10 (public) vs 14 (graph) surgeon count — extra ~4 = broader UGI/general cohort.
- DM'd Eli the full map 2026-06-28 (5-part Discord): headline + graded hypothesis + 3 crowns + camps
  + "no fabricated feuds" caveat + bottom line (Magdy = softest BD wedge).

## Key learnings (run additions)
- **Concurrency:** ≤6 research agents is the stable ceiling (8 ok, 10 stalled). Pipeline =
  launch ≤6 locked agents → `harvest.py` pulls final JSON from `.output` transcripts
  (out-of-context) → `augment.py` scrapers → `write.ts` (idempotent delete-then-insert).
- **Agent fetch cap is critical:** uncapped agents stalled the 600s watchdog on slow/blocking
  sites (one zombie ran ~1.5h). Capping agents at ~13-15 web fetches dropped runtimes to
  ~3 min and eliminated stalls.
- **ScheduleWakeup backstop** every batch makes the loop self-sustaining/crash-recoverable;
  DB `enrichedAt` is the source of truth, harvest always re-filters to null.
