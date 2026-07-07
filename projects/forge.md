# FORGE — training app with a visual layer (iOS)

Eli's brief (2026-07-07, Discord, verbatim file in inbox): FORGE is a training app — log reps/weights — whose
differentiator is the **visual layer**:
1. **v1 core: phone-camera movement tracking.** Record a lift, derive the body's movement, diagram it, and
   layer on analysis — e.g. "bench press: left elbow was uneven / under more tension." Rep positioning, form.
2. **Phase 2: body scanning.** ~4 photos (front/back/sides) → 3D body model → size/proportions → volume →
   mass estimate; iPhone LiDAR validates scale; ambition to calibrate against DEXA.
3. **Parked (Eli's own call): gym evolution.** Gyms sign on, security cams → 3D gaussian-splat environment,
   member just trains and sees it on their phone; can correct miscounted reps/weights. Also curious about
   phone-side splats (ultrawide+main+LiDAR corroboration). Research-grade — NOT the focus.
- **Success criteria: a deliverable iOS app.** iOS-first for simplicity; later a single-camera Android version.

## MOMO's assessment (sent 2026-07-07)
- Core v1 is genuinely buildable: **Apple Vision framework 3D body pose** (`VNDetectHumanBodyPose3DRequest`,
  iOS 17+, 17 joints, on-device, ordinary camera) covers exactly the joint-angle/asymmetry/rep-counting/
  bar-path/tempo analysis FORGE needs. On-device = private (video of bodies — matters).
- Build order argued: v1 = logging + camera form analysis; body scan = phase 2; splats/gym = parked.

## Blockers / asks (sent to Eli, awaiting)
1. **Xcode NOT installed on the mini** (verified — only CommandLineTools). ~12GB Mac App Store install,
   needs his Apple ID. Blocker for any build.
2. **Apple Developer account** — $99/yr only needed at TestFlight/distribution time; free Apple ID suffices
   for dev builds onto his own phone. Money decision = Eli's, deferred.
3. **Which iPhone does Eli have?** Real device required (simulator has no camera); LiDAR = Pro/Max only.

## Research in flight (3 parallel agents, launched 2026-07-07)
1. iOS pose-estimation stack: Vision 3D pose vs MediaPipe/MoveNet/QuickPose etc., rep-counting + form-fault
   methodology, bar-path tracking, multi-cam + LiDAR fusion reality, occlusion problems (bench = horizontal body).
2. Competitive landscape: form-analysis apps (Tempo/Asensei/Kemtai/Exer etc.), logging incumbents (Strong/Hevy),
   VBT bar-path apps, gym-camera prior art, where the gap is + lessons from failures.
3. Body-scan tech: ZOZOFIT/Bodygram/MeThreeSixty/Spren etc., SMPL-fitting accuracy from ~4 photos, **SMPL
   commercial-licensing gotcha**, LiDAR's real contribution, build-vs-license.
→ Results to be distilled into an architecture + roadmap here.

## Research findings

### 1. Body scan (agent 3 of 3, returned 2026-07-07) — PHASE 2 DECISION BRIEF
- **Circumference measurements from 2-4 photos: credible for TREND tracking** (real-world RMSE 2.5-6.1cm per
  peer-reviewed MeThreeSixty study), not clinical claims. Tight clothing is LOAD-BEARING, not optional.
- **Body-fat % from photos: best-in-class is Spren** — MAE 2.6%, r=0.95 vs DEXA (Pennington study) — but ALL
  validations were lab-conditions; assume real-world MAE 3-5%+. Beats BIA scales though.
- **⚠️ Eli's volume→mass derivation: NO independent validation exists anywhere.** Weakest link — frame as
  experimental until we run our own study. (Pushback to deliver.)
- **⚠️ LiDAR is marginal, not transformative**: nails height (0.55% err) but waist error still 6.90% — one
  viewpoint can't see the back. Honest value = absolute SCALE (replaces reference-object), not circumference
  accuracy. mPort's pod→phone pivot got WORSE per users.
- **⚠️ SMPL licensing landmine**: SMPL/SMPL-X/STAR academic license bans ALL commercial use INCLUDING training
  models on it. Commercial = paid Meshcapade license (price on request). The CC-BY "SMPL-Body" carve-out strips
  the shape blendshapes = useless for measurement. Bodygram/Spren sidestepped via proprietary regressors + own
  datasets (expensive).
- **Recommendation: LICENSE for phase 2** — Bodygram Platform (2-photo API, clinic deployments, real dev docs)
  is the closest fit; Spren has the best accuracy data but no public SDK (sales inquiry).
- **Honest marketing ceiling**: "track change over time under repeatable conditions" — never "medical-grade";
  BF% always caveated, never diagnostic (avoids TGA/FDA device framing). Amazon Halo died partly from privacy
  backlash over near-nude scans — privacy posture matters.

### 2. iOS pose stack (agent 1, returned 2026-07-07) — V1 ARCHITECTURE BRIEF
- **Recommended stack: Apple Vision, tiered.** `VNDetectHumanBodyPoseRequest` (2D, 19 joints, iOS 14+,
  video-frame-rate, universal baseline) + `VNDetectHumanBodyPose3DRequest` (17 joints IN METERS, root-origin +
  cameraOriginMatrix, iOS 17+) layered where available. LiDAR consumed opportunistically — NOT required.
  Free, on-device/private, Neural-Engine optimized. ⚠️ Single-person only — locks onto nearest person
  (gym bystanders/mirrors are a real failure mode).
- **Bar path = a SEPARATE pipeline**: Core ML YOLO-style barbell detector (Roboflow datasets + OSS prior art
  exist) seeded into Vision's free `VNTrackObjectRequest`. This is what shipped VBT apps do.
- **Rep counting**: peak/valley detection w/ prominence+hysteresis on the primary joint angle (elbow=bench,
  knee/hip=squat). OSS prior art. v2 upgrade path: viewpoint-invariant method arXiv:2107.13760 (MAE 0.06 reps).
- **Skip for v1**: ARKit body tracking (AR-session overhead, 91 joints mostly interpolated, no accuracy edge);
  AVCaptureMultiCamSession+LiDAR fusion (thermal throttling 30→15fps documented, Pro-only, research-grade —
  later "Pro mode" at best).
- **⚠️⚠️ THE structural finding: bench press is the HARDEST case.** (a) Horizontal/lying poses degrade ALL
  pose models (trained on upright data — dataset bias, switching SDKs won't fix); (b) the bar passes over
  chest/face = structural occlusion; (c) 2024 PLOS ONE validation: MyLift missed 84% of bench reps; Metric
  acceptable only on bench w/ bias; **Qwik VBT matched Vicon gold standard** — so bench is best served by
  BAR-PATH analysis, standing lifts (squat/deadlift/OHP) by body pose. Shape v1 exercise coverage accordingly.
  This reframes Eli's flagship "bench elbow" example — deliverable, but via bar path + lower-confidence joint
  fallback, not naive full-skeleton tracking. PUSHBACK TO DELIVER.
- **No independent accuracy benchmarks exist for Apple's pose APIs** — in-house validation needed before
  shipping form-fault claims. Alternatives ranked: MediaPipe BlazePose (33 landmarks, only if needed),
  QuickPose ($25-1000/mo, just wraps MediaPipe), MoveNet (no edge on iOS), Kemtai/Asensei (B2B, not indie-viable).

### 3. Competitive landscape (agent 2, returned 2026-07-07) — MARKET BRIEF
- **The gap is REAL and unserved**: market split into two non-overlapping tribes — logging apps (Strong/Hevy/
  Boostcamp/Jefit: beloved logging, ZERO camera investment, their AI budget goes to program generation) vs
  camera-form apps (Onyx/Zing/Gymscore/Kemtai + Tempo/Tonal/OxeFit hardware: real-time CV, no serious-lifter
  logging). Nobody does both. Risk framing: unserved because doing both well is hard.
- **Hardware form products are dying**: Peloton Guide discontinued Jul 2025 ($495→free death curve); Tempo
  stalled ($298M raised, no round since ~2021); Tonal layoffs. **Phone-only is the right structural bet** —
  but must earn accuracy Tonal gets from fixed hardware.
- **THE trust lesson (echoed everywhere)**: every failing app claims "any exercise" and underdelivers (Onyx:
  4.9★ store vs "worst calibration" on independents — review-gap red flag; Zing 4.8 vs 3.5 Trustpilot).
  Users forgive manual setup if the number is trustworthy; they NEVER forgive unreliable automatic claims.
  → **Ship a NARROW validated exercise list, exercise-by-exercise, honest in-app about coverage.**
- **Bar-path VBT**: phone CV can match $1000+ hardware (Qwik = Vicon-grade) but "automatic + real-time +
  reliable" is the unsolved trifecta — each shipped app has 2 of 3. FORGE's opportunity in bar tracking.
- **Retention drivers**: fast logging, program structure, objective progress numbers (things that pay EVERY
  session). Churn drivers: gross-error-only feedback, unreliable rep counts, mid-set camera repositioning,
  billing tricks. Logging = table stakes (can't out-log Strong/Hevy as the wedge); camera trust = the wedge.
- **Gaussian-splat gym capture: genuinely open, zero incumbents** (Perch is enterprise team-sports bar
  tracking). Eli's long-term instinct has no competitor — and no demand proof yet. Correctly parked.
- Table stakes for any entrant: fast set/rep/weight logging + plate math + rest timers, program templates w/
  progressive overload, Apple Watch companion, 1RM/volume/PR analytics.

## Notes
- NV Health remains the operational priority for open threads (GTM conversion publish-state check + secure PDF).
- FORGE is greenfield — nothing committed yet; no repo created yet.
