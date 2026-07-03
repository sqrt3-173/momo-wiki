# Engine hardening & adoption plan — 2026-07-03

Standing assessment from Eli's deficiency question + the agent-orchestration landscape report
(inbox item 10). Context that reframes everything: **business data enters the engine soon** —
Eli's stated intent (2026-07-03) is the engine live this week, then "slowly bring in business
data for it to act on." Ordering below is gated on that.

## INCIDENT 2026-07-03 ~19:20 — model-limit blackout (deficiency #2 bit live)
Everything all day ran on **Fable 5**, subagents included (subagents inherit the session model).
Two parallel phase-2/3 website planners ran ~75 min each and hit the **Fable 5 account usage limit**,
dying without producing plans. Worse: the **main loop was also Fable 5**, so once the limit hit MOMO
could not send Discord either — the bridge looked dead for ~45 min while Eli sent "hello???". Eli
switched the session to **Opus 4.8** to unblock (separate limit pool). No data lost; the heartbeat
survived (idle all evening, never spawned a session, never hit the limit).
**Two fixes this exposed:**
- **Model tiering (deficiency #2), now mandatory, not optional:** heavy judgment (planning, checking,
  hard triage) on a frontier model; mechanical bulk (code execution, extraction, doc edits) on
  **Sonnet**; never run a whole fan-out on one limited model. Set per-agent via the Agent `model` param.
- **Bridge-mute-on-limit (new, deficiency #6):** an account-wide model limit takes down MOMO's ability
  to even report it's stuck. Harden: a fallback model for the main loop / a limit-detecting watchdog
  that DMs Eli "hit the X limit, switch me" rather than going silent. Unverified whether
  `--fallback-model` covers usage-limit (vs overload) — test before relying on it.

## The five deficiencies, with implementation shape

1. **Fresh-context output verification** *(adopt first — the MAST evidence says verification is
   where autonomous systems structurally fail).* After a background session lands judgment work
   (triage fates, wiki filings), a reviewer agent sharing ZERO context with the worker reads only
   the receipts + goals pages and votes confirm/challenge. Challenges → Eli's queue or re-run,
   never silent acceptance. Verify everything first, sample once the challenge rate is known.
   Substrate: a `verifications` receipt table + either a `review` tick unit or an in-tick
   fresh-subagent pass. **Must land before business data.**

2. **Model tiering.** `--model` per unit type in the tick wrapper; `plans.model_tier` already
   exists. Caveat: triage judgment is the product — one A/B (same batch, cheap vs frontier,
   compare fates) decides which units downgrade. Baseline comes from the first digest. Note the
   ~$0.36 session boot floor is context volume; a cheaper model shrinks the multiplier, not the
   token count.

3. **Ranked approval queue.** Digest orders decisions by blast radius (irreversible/money/public
   → roadmap-shaping → routine batched), criteria written down (reuse the guard's risk
   categories); plus a standing open-decisions wiki page so nothing rots in chat scroll.
   Directly raises the system's true ceiling: Eli's review capacity per hour.

4. **Isolation (the Trinity question).** Verdict: **steal the pattern, skip the platform** —
   Trinity's queue/scheduler/audit duplicate our engine; adopting it wholesale migrates the
   brain into a ~7-month-old runtime and abandons the judgment layer nothing off-the-shelf has.
   Isolation layers in cost/value order for OUR risk profile:
   1. **Scoped DB roles** — worker sessions get a login with zero grants on business databases
      (physically cannot, not trusted-not-to). Cheap, this week.
   2. **Egress control** — the guard honestly cannot catch in-process network calls
      (node/python); before patient/customer data is touchable, worker-session outbound needs a
      wall. Prototypes exist: `ops/egress_proxy_test.py`, `ops/egress_policy.py` (verify current
      state before relying on them).
   3. **Containers** — real isolation, real plumbing; honest catch: a containerized session
      loses macOS keychain OAuth → forces API-key billing, a new spend surface needing Eli's
      approval. Third layer, when blast radius demands it.

5. **Parallel reads (LOOP-05, parked).** Doctrine: fan out reads, serialize writes (parallel
   writers fail via decision fragmentation; reads verified 90.2% better in fan-out). In-session
   fan-out already used everywhere. Fleet-level: tag units read-safe vs write; writes stay
   single-file; read units run N sessions (claiming is already race-safe). Trigger, on data:
   queue depth consistently outrunning what serial clears overnight — AND only after #3 exists,
   or parallelism just floods the review queue.

## The order (updated post-incident + NV deadline)
0. **NV Health website — HARD DEADLINE Sun 2026-07-06 10:00** — the immediate priority; everything
   below is sequenced around it. See [[../projects/nv-health-website]].
1. **Model tiering (#2)** — status: **TACTICALLY LIVE** (deadline execution agents forced to Sonnet
   via the Agent `model` param; frontier reserved for planning/checking). **Permanent wiring TODO
   right after the deadline:** default tiers baked into the tick launch (`momo-tick.sh --model`) and
   the standard spawn helpers, per unit kind — not hand-set each call. Needs the cheap-vs-frontier
   triage A/B to set which units are safely Sonnet.
2. **Bridge fallback (#6)** — status: **DOCUMENTED, NOT BUILT.** A model-limit watchdog: on a
   usage-limit error, the main loop DMs Eli "hit the <model> limit — switch me" instead of going
   mute, and/or auto-falls-back to another model for the bridge. Verify `--fallback-model` behaviour
   under usage-limit (vs overload) first. **Do right after the deadline** — it's the difference
   between a 45-min blackout and a one-line heads-up.
3. Then: **#1 fresh-context verification + #3 ranked queue** → **#4.1 scoped DB roles + #4.2 egress
   wall** → business data enters, gated on those → **#4.3 containers / #5 parallelism** when data
   demands.

*Linked from [[momo-engine]]. Source: Eli's questions 2026-07-03 + inbox item 10 (landscape report,
adversarially verified) + the 2026-07-03 model-limit incident above.*
