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

## Notes
- NV Health remains the operational priority for open threads (GTM conversion publish-state check + secure PDF).
- FORGE is greenfield — nothing committed yet; no repo created yet.
