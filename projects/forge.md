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

## BUSINESS STRATEGY / POSITIONING (Eli, 2026-07-07)
FORGE is fundamentally **a tool to facilitate COACHING**. Two go-to-market angles:
1. **B2C** — consumers/gym-goers use it themselves for **autonomous improvement** (solo self-coaching).
2. **B2B2C** — **coaches + gyms** use it as a professional tool. Vision: become **the industry standard for
   what's expected in programming**, and a **selling point for gyms** (a gym offers FORGE → competitive edge).
- **Strategic implications for the build** (why this matters, not just a label):
  - The **deep biomechanics insight layer is now strategically CENTRAL, not a nice-to-have** — coaches need
    pro-grade analysis (joint loads, velocity/power, technique, asymmetry) to justify it as a professional tool
    + "industry standard for programming."
  - **Multi-user / roster features** become real: coach ↔ athletes, shareable 3D replays + stat reports (a
    coach reviews an athlete's lift remotely + prescribes), a coach dashboard, programming tools atop the log.
  - The **parked gym-camera era fits the gym B2B angle** directly (install cams → members get auto-capture →
    gym selling point). The dynamic-gaussian-splat "gym install" isn't just a cool future — it's the B2B hook.
  - **Programming** (already in the core: templates + progressive overload) is load-bearing for the coach angle.
- Sharpens priority: build the insight depth + the eventual coach/roster + sharing layer as first-class, since
  the B2B2C "industry standard for coaching/programming" is a core wedge, not an afterthought.

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

## Architecture + repo (2026-07-07)
- **Repo: `/Users/momo/momo/projects/forge/`** (git init'd, local-only — GitHub push needs Eli's OK).
  **`ARCHITECTURE.md`** = the v1 system design: SwiftUI/SwiftData, iOS 17+ min, ForgeLog (logbook) +
  ForgeVision (Vision 3D pose + YOLO bar tracker + rep engine, all on-device) + ForgeReplay (skeleton/bar-path
  overlay replay — replay-first, NOT real-time coaching), 5 validated launch lifts (bench via bar-path),
  privacy-as-feature, milestones M1-M5. momo_work project 10, phases 1-5 created.
- **Eli's device: iPhone 17 Pro** (best case — top Neural Engine, LiDAR, ProMotion).
- **Sole blocker: Xcode install on the mini** (Mac App Store, Eli's Apple ID, ~12GB). Asked; awaiting.
- **v1.5 = ERGS (Eli, 2026-07-07)**: SkiErg/RowErg/BikeErg — camera technique + **PM5 BLE sync** (published
  official Concept2 spec; stroke-by-stroke power + force curve). Technique→performance correlation with
  machine ground truth = the novel product. Ergs are camera-favourable (side-on, cyclic, no occlusion).
  ⚠️ Echo bike likely closed console (verify). Details in projects/forge/ARCHITECTURE.md §v1.5.

## M1 FIRST LIGHT (2026-07-07 ~11:20) — app builds + runs
- **Toolchain**: Xcode 26.6 (macOS 26.5). Eli did the sudo steps (guard allowlist +6 xcode tools — guard file
  is root-owned + uchg-immutable, needs `sudo chflags nouchg` to edit; xcode-select→Xcode.app; licence;
  runFirstLaunch). I pulled the iOS 26.5 platform (8.5GB, `xcodebuild -downloadPlatform iOS`).
- **First build SUCCEEDED first try** — hand-authored pbxproj (objectVersion 77, PBXFileSystemSynchronizedRootGroup,
  shared scheme, bundle id `agency.catapult.forge`, SWIFT_VERSION 5.0, iOS 17 min) works headlessly.
- **Running on iPhone 17 Pro simulator** (device UDID 665D0D31-…; build → `xcrun simctl boot/install/launch`,
  screenshot via `simctl io screenshot`). Screenshot sent to Eli.
- M1 contents: SwiftData logging core (Workout/ExerciseEntry/SetRecord), catalog w/ AnalysisProfile camera
  badges (5 lifts), quick-add sets, rest timer, plate math, history + e1RM PRs. Phase M1 = **VERIFIED** (UI-test target hand-wired into pbxproj; e2e flow test PASSING — clean-slate via -uitest-reset in-memory store; 4 iterations: sheet-animation race, my-wrong-Epley-check (app right), launch-timing, state pollution);
  remaining: end-to-end flow test in sim → then device install (needs Eli's Apple ID in Xcode, 2 min).

## M2 + M3 progress (2026-07-07, device era)
- **M2 shipped to Eli's iPhone 17 Pro**: live camera + skeleton overlay. His feedback drove:
  - Camera flip (front/back) + lens picker (0.5× ultra-wide / 1× wide). ⚠️ Front cam mirrored → fixed.
  - **Camera freeze+black bug (MY regression)**: was configuring AVCaptureSession on the MAIN thread →
    UI hang. Fixed: all session config + start/stop on a dedicated sessionQueue, frame delegate on a
    separate videoQueue, nonisolated session/engine/queues. LESSON: never touch AVCaptureSession on main.
- **Device signing wall**: headless `xcodebuild` for the DEVICE fails ("keychain write permissions" —
  the cert Xcode created is GUI-locked). So device deploys need Eli's ▶ press for now. Simulator builds +
  ALL tests run fully headless. TODO: dedicated build keychain / set-key-partition-list for headless deploy.
  Team ID: **4PR8AVQYY7** (Eli's non-personal dev-program account). Bundle: agency.catapult.forge.
- **M3 analysis (executing)** — all verified HEADLESS (no camera/human needed = the autonomy unlock):
  - **Kinematics**: joint angles (knee/elbow/hip) + L/R asymmetry (the "uneven elbow" read).
  - **RepEngine**: streaming rep segmentation, per-rep ROM/tempo/bottom-angle, hysteresis + false-dip
    rejection. Unit tests CAUGHT 2 real bugs (ROM missed the top band; timing) → fixed.
  - **1€ filter smoothing** (PoseSmoother): kills joint wobble = much of Eli's "points inaccurate".
  - **SubjectTracker**: locks largest/closest person, IoU continuity, grace re-lock, gates pose to subject
    box → bystanders can't steal the skeleton (Eli's "someone gets in frame").
  - **ForgeTests unit-test target** hand-wired into pbxproj (TEST_HOST/BUNDLE_LOADER). 24 tests green.
  - Test target pattern for the hand-authored pbxproj now proven twice (UI + unit).
- **Honest 3D framing (told Eli)**: a 3D SKELETON (VNDetectHumanBodyPose3DRequest) = buildable, real
  accuracy jump. A photoreal 3D body MESH real-time from one camera = NOT viable today (= body-scan phase,
  SMPL licensing). Didn't overpromise on "Fable 5 can build anything."
- **Eli's clarified vision (2026-07-07)**: NOT photoreal — a tracked 3D skeleton you RECORD, then watch
  REPLAY in an orbitable 3D environment, with a full pro-athlete stat suite (TUT, ecc/con, ROM, asymmetry,
  fatigue). "Most comprehensive app ever." + asked re LiDAR.
- **DELIVERED (M3, all verified, 19 tests green)**: record→3D-replay→stats.
  - `Pose3DEngine` (VNDetectHumanBodyPose3DRequest — 3D joints + 2D proj + depthAssisted flag + bodyHeight;
    3D API compile-verified against SDK: `.position` simd_float4x4, `pointInImage`→VNPoint.location,
    `heightEstimation == .measured` = LiDAR-assisted). LiDAR is opportunistic-automatic on the 17 Pro.
  - `Kinematics3D` (camera-angle-invariant 3D angles), `StatsComputer` (per-rep TUT/ecc/con/ROM/asymmetry +
    set fatigue%), `SessionRecorder` (NSLock frame bank on videoQueue), `SkeletonSceneView` (SceneKit
    orbitable skeleton on a floor), `ReplayView` (scrub + stat sheet + LiDAR badge), record button in capture.
  - LiDAR honest answer given: helps HERE (scale/real-units + Apple already uses it), one-viewpoint limit.
  - ⚠️ Still needs Eli's ▶ + on-device visual verification (SceneKit skeleton orientation/scale, live 3D fps).
- **PERF pass (2026-07-07)**: reuse VN request objects (never per-frame), person-detection every 8th frame,
  throttle pose ~20fps + 3D ~12fps. Good hygiene — but was NOT the crash cause (I guessed; Eli pushed back).
- **THE camera crash — real root cause (from DEVICE CONSOLE, not guessing) 2026-07-07**:
  - Symptom: camera ~20s to open then iOS killed the app ~30s.
  - Diagnosis method: `xcrun devicectl device process launch --console` gave nothing (app has no logging);
    the winner = Eli ran via **Xcode ▶ (debugger attached)** + screenshotted the debug console. Log showed
    `Attempt to present <PresentationHostingController> ... which is already presenting` REPEATED + then
    `FigCaptureSourceRemote Fig assert err=-17281` repeated (camera source failing over and over).
  - CAUSE: `.fullScreenCover` was attached INSIDE each List row (`ExerciseSection`). SwiftUI recycles rows →
    present/dismiss loop → a NEW AVCaptureSession spun up + torn down repeatedly → camera never stabilises →
    slow/black → watchdog/camera-failure kill.
  - FIX: ONE top-level `.fullScreenCover(item: $cameraTarget)` on ActiveWorkoutView; row's camera button
    just sets `cameraTarget` (Identifiable wrapper) via a closure. One session, opened once. M1 test green.
  - LESSONS: (1) NEVER attach `.fullScreenCover`/`.sheet` inside a recycling List/ForEach row — present at a
    stable top level. (2) When blind on a device bug, STOP guessing → read the actual console (Xcode ▶ +
    debugger screenshot is the fast path when headless deploy is keychain-locked).
  - ⚠️ Headless device deploy still blocked (login-keychain private key inaccessible to background xcodebuild;
    even without -allowProvisioningUpdates: "no signing cert with a private key"). Eli's ▶ needed per deploy
    until a keychain-unlock is set up.
- **Sub-momo pilot**: Eli wants 2-3 parallel instances, same Claude account, own Discord channels. NUNU
  templates = the kit. Proposed pilot: FORGE-momo owning FORGE in #forge, MOMO as director. Awaiting go.

## MOONSHOT vision (Eli, 2026-07-07) + honest staging
Eli wants far more than a skeleton: an **anatomically accurate 3D BODY MESH of himself** reconstructed from
video (not a stick figure); his own body shape captured + motion-driven; **LiDAR + multi-cam stereo +
monocular depth fusion** for accuracy; **skeleton on the video itself**; and a **gaussian splat** to explore
the scene. "I want it to be perfect."
- **MOMO's honest tiers (told Eli, refused to yes-man "perfect")**:
  - NOW: skeleton-on-video ✅ DELIVERED; anatomical body MESH (SMPL-family, has commercial-licence gate).
  - HARD-but-real: depth fusion (LiDAR+stereo+monocular); personalized body shape scan + motion drive.
  - FRONTIER: gaussian-splat scene exploration — real tech, nobody's shipped it for fitness, live-on-phone
    isn't there; offline-from-video heavy. Genuine R&D bet, not a sure thing.
  - Won't promise "perfect" — promised best-achievable-on-iPhone + honest ceiling at each step.
- **3 deep-research agents launched (2026-07-07)**: (1) video→3D human mesh (HMR2/4D-Humans/SMPLer-X,
  on-device CoreML, SMPL licensing, personalization, Apple-native mesh?); (2) iPhone depth fusion
  (LiDAR sceneDepth + AVCaptureMultiCam stereo + Depth Pro/Depth Anything monocular, synchronizer gotchas,
  honest per-signal contribution); (3) gaussian splatting on iOS (3DGS/4D-dynamic/human-GS, on-device vs
  cloud, Luma/Polycam/Scaniverse, maturity verdict). → synthesize into a technical plan + honest verdicts.
- **DELIVERED now**: `VideoRecorder` (AVAssetWriter full-rate video during recording) + `VideoSkeletonView`
  (AVPlayer + Canvas skeleton overlay on real footage, time-synced) + ReplayView On-video/3D toggle.
  stopRecording now async. 19 tests green.
- ⚠️ Front-camera mapping bug (orientation/mirroring) — acknowledged secondary; back-camera propped is the
  real workflow.

## Moonshot research findings

### Gaussian splatting (agent returned 2026-07-07) — HONEST VERDICT
- **"Fly around your dynamic lift as a gaussian splat from one phone" = NOT achievable, not close.** Physics,
  not effort: one camera = one viewpoint/instant; free-viewpoint of FAST DYNAMIC content needs synchronized
  MULTI-camera rigs (DyNeRF=21 cams; commercial volumetric studios 10-100+). Every shipped GS product
  (Luma/Polycam/Scaniverse/KIRI) is STATIC-scene only + says motion blur/moving people = artifacts.
- Dynamic/human GS (GaussianAvatar/HUGS/Dynamic Gaussian Marbles) = research-grade, per-subject mins-hours,
  strips the scene (avatar only), collapses on fast/occluded motion. Not product-ready.
- On-device: STATIC splat training ships (Scaniverse, ~1-2min on-phone); rendering static splats on iOS is
  SOLVED (MetalSplatter, MIT). NO on-device dynamic/4D training exists anywhere.
- **Shippable reframe (the real feature hiding in the idea)**: a **"3D PR pose" trophy** — hold lockout /
  top-of-squat, slow phone orbit (static), run the proven static pipeline (Luma/KIRI API or on-device
  Scaniverse-style) → render with MetalSplatter. Delivers "explore yourself in 3D" of a HELD pose. Buildable.
- Full moving-lift = a gym multi-camera install + DyNeRF pipeline (the parked "gym era" phase), NOT a phone
  feature. Recommendation: don't roadmap dynamic-splat for v1/v2; don't license (no vendor sells it).
- ⚠️ Volograms "Volu" = closest shipped (1 phone, moving person → neural hologram) but NOT gaussian splat,
  scene-stripped, lower fidelity. Honest state of the art for single-phone moving-person-in-3D.

### Depth fusion (agent returned 2026-07-07) — ACCURACY STACK VERDICT
- **Recommended stack**: LiDAR `sceneDepth` (metric SCALE anchor) + **multi-frame ROTATION capture** (the
  biggest real accuracy win) + SMPL/SMPL-X shape-prior fit (solves occlusion) + monocular Core ML refiner
  (Depth Anything V2-small, ~31-34ms on-device, scale-locked to LiDAR). 
- **⚠️ PUSHBACK on Eli's multi-camera/binocular idea: SKIP dual-cam stereo (wide+ultrawide).** iPhone camera
  baseline is tiny (~1.4cm) → useless disparity at 2-3m body distance; wide+ultrawide isn't even a supported
  stereo pair (FOV mismatch); Apple's calibrated stereo is wide+TELE only. No product/study does usable
  dual-phone-cam body measurement. Costs thermal+engineering for ~zero gain. The "binocular cues" instinct
  doesn't survive the physics of the small baseline.
- **Per-signal ranking**: LiDAR=HIGH (scale, ~1-3cm gross <3m, weak on circumference/occlusion as predicted);
  multi-frame rotation=HIGH (Prism Labs: waist ±3.4cm R²=0.97, hip ±1.7cm — DXA-validated, beats LiDAR+ML
  fusion's 8% err); SMPL fit=MED-HIGH (occlusion); monocular=MED (refiner only, must scale-lock); dual-cam
  stereo=SKIP.
- **Ceiling**: ~1cm skeletal/height, ~1.5-3.5cm circumference (with the stack; 4-8% if LiDAR-single-shot
  only). Composition (BF%/volume) = trend-tracking, NOT DXA-diagnostic. Market accordingly.
- API: stay in ARKit (`sceneDepth`, timestamp-aligned, no manual synchronizer needed); raw `sceneDepth` not
  `smoothedSceneDepth` for per-instant geometry; `isFilteringEnabled=false` for measurement.
- Recurring theme (both this + mesh): the win is the **shape-prior fit + multi-view geometry**, NOT more
  sensor hardware. SMPL keeps coming up → licensing gate is the pivotal question (mesh agent).

### Video→body mesh (agent returned 2026-07-07) — THE CENTERPIECE VERDICT
- **A 3D body MESH of yourself, moving, IS buildable.** Two real paths:
  - **A — License Meshcapade**: they SHIP exactly this ("Meshcapade Me": phone photo/video → personalized
    SMPL avatar w/ real measurements, marketed at FITNESS). Real, legal, personalized-to-you. Cost = custom
    enterprise quote (sales@meshcapade.com, no public pricing — must ask). Fastest to a genuine product.
  - **B — Apple-native**: ARKit gives a FREE real-time on-device skeleton (no mesh); drive a custom-authored
    (non-SMPL) rigged mesh scaled per-user via height/proportion sliders. Zero licensing risk, cheaper, but
    coarse (generic body scaled, not true shape reconstruction).
  - **C — port open HMR model to Core ML yourself**: NOT recommended — still needs Meshcapade asset license
    (licensing attaches to the MESH ASSET, not the code — even ARKit-driving-SMPL needs it) + months of
    unproven porting. 
- **⚠️ THE licensing gate**: SMPL/SMPL-X/STAR/SUPR all Max Planck NON-COMMERCIAL. Commercial = pay Meshcapade
  (exclusive sub-licensor). NO permissive full-body parametric mesh exists. GHUM Apache-2.0 only covers 33
  landmarks NOT the mesh (not an escape). This is a real money decision.
- **⚠️ Apple gives skeleton, NEVER a mesh** (Vision + ARKit both confirmed joints-only). Real-time mesh
  recovery (HMR2/GVHMR/Multi-HMR) is GPU-only; NO Core ML port exists publicly → ~15-30fps on A18/A19 is a
  plausible ESTIMATE after months of R&D, not proven.
- **⚠️ Honest ceiling on "anatomically accurate"**: = plausible + proportion-correct, NOT clinical. 5-9cm
  error even in labs, worse in a gym (clothing/occlusion/extreme poses). SMPL shape space underrepresents
  shredded/very-large builds → "best fit within average-body space," not true muscle reconstruction. MUST
  calibrate Eli's "perfect" expectation.
- **Recommendation: get the Meshcapade quote FIRST** (resolves the cost unknown). If affordable → path A. If
  not → path B ships something real at lower fidelity. Rendering (SceneKit SCNSkinner, ~7-10k verts) = trivial,
  NOT the bottleneck; the bottleneck is upstream per-frame pose.

### ⚠️ CORRECTION (Eli caught, 2026-07-07): Meshcapade is DEAD (Path A gone)
- Eli went to the Meshcapade site → **"Meshcapade is now part of Epic Games. As of April 18 the Meshcapade
  online platforms have been shut down."** The mesh-agent's research was STALE on this.
- **Path A (license Meshcapade Me / their API) = DEAD.** SMPL commercial licensing now sits with Epic
  (murky during acquisition). Epic has MetaHuman (Unreal-Engine digital humans — wrong tool for a native
  iOS fitness app, too heavy).
- **Consequence: Path B (Apple-native, free, own-it, no external dependency) is now the clear route** — no
  licensing limbo, no reliance on a vanished vendor. Build our own body pipeline on ARKit's free real-time
  skeleton. (Verification agent launched: confirm Epic/SMPL status + any alternative body-mesh SDK provider.)
- LESSON: research can go stale fast on fast-moving vendors; Eli poking the actual site caught it. Verify
  vendor status before recommending a license path.

### ✅✅ VERIFICATION RESULT (2026-07-07) — a BETTER path emerged
- **Meta released MHR (Momentum Human Rig) + SAM 3D Body on 2025-11-19, Apache-style PERMISSIVE COMMERCIAL
  license, open on GitHub** (facebookresearch/MHR; SAM 3D Body code+weights on GitHub/HF).
  - **SAM 3D Body = single photo → personalized 3D body MESH** — literally what Meshcapade Me did — OPEN,
    self-hostable, NO licensing negotiation, NO fee, commercial-OK.
  - MHR = the parametric skeleton+mesh+blendshapes rig (multi-LOD) it targets.
- **This is the new Path A, and it's STRONGER than licensing SMPL ever was**: own the pipeline, zero vendor
  dependency, zero fee. Appeared months before Meshcapade shut down — covers the gap perfectly.
- Confirmed: SMPL commercial licensing moved to **Max Planck Innovation** (smpl@max-planck-innovation.de) —
  slow/bespoke/no pricing; NOT Epic. Avoid. Ready Player Me shut down (Netflix, Jan 2026). Presize→Meta 2022.
  Alt SDKs still live: in3D (real mesh, opaque pricing), Avatar SDK/MetaPerson (pricing tiers) — fallbacks.
  Also: Anny (Naver, Apache-2.0) open body SHAPE model (not a photo-fitter).
- **THE PLAN (updated)**: build FORGE's "Meshcapade Me" on **Meta MHR + SAM 3D Body** — onboarding photo →
  SAM 3D Body → user's personalized MHR body mesh; Apple's free real-time skeleton drives it moving during
  workouts; SceneKit renders. Photo→mesh likely a cloud/on-device inference step (R&D); NO licensing wall.
  ⚠️ Still a real engineering build (integrate Meta models, probably a cloud inference step), but the BLOCKER
  (licensing) is gone. Recommend this as the path.

## SYNTHESIS → the honest FORGE visual-layer plan (all 3 research streams)
1. **3D body mesh of you** = the centerpiece, buildable. Decision gate = Meshcapade quote (path A) vs
   Apple-native free/coarser (path B). Ceiling = proportion-correct, not clinical.
2. **Accuracy** = LiDAR (scale) + multi-FRAME rotation capture + SMPL fit + monocular refiner. **SKIP
   multi-camera stereo** (Eli's binocular idea — physics of the tiny baseline kills it). ~1-3.5cm ceiling.
3. **Gaussian splat** = "3D PR pose" static trophy (buildable) NOW; dynamic fly-around = gym-camera era (parked).
4. Cross-cutting theme: the win is the **shape-prior fit + multi-view geometry**, NOT more sensor hardware.
5. Shipped already: skeleton-on-video ✅. Next buildable-now: the Apple-native body mesh (path B) as a
   no-licensing-risk start while the Meshcapade quote is pursued.

## M6 BUILD STARTED (Eli's GO, 2026-07-07)
- **Phase order**: (1) retargeting math ✅ → (2) onboarding body-scan capture (guided 360 + LiDAR) →
  (3) body mesh render+drive (placeholder body first, sim-screenshot verified, then real MHR) →
  (4) cloud SAM 3D pipeline.
- ✅ **PoseRetargeter** built + 25 tests green: Apple 17 joint POSITIONS → bone ROTATIONS (quaternions) to
  drive a rigged mesh; handles parallel/antiparallel edge cases (naive cross-product NaNs there). Foundational,
  not throwaway; calibrates to MHR bind pose when the mesh lands. Fully headless-verified.
- **Discipline note**: did NOT build the body-render blind (SceneKit isn't headless-verifiable without a
  sim-screenshot harness). Next: build the mannequin render + a sim harness to screenshot-verify it, THEN the
  scan-capture flow (needs device camera+LiDAR). Verify-what-I-build holds even at the frontier.

## Notes
- NV Health remains the operational priority for open threads (GTM conversion publish-state check + secure PDF).
