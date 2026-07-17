# Incident: local Postgres down — momo_work unreachable (2026-07-14)

**Severity:** high (blocks the whole tick engine, not just one unit). **Outcome:** open —
needs a manual restart only Eli or an interactive session can do; headless ticks cannot
self-heal by design (guard correctly blocks the tools needed). **148 ticks have now hit
this identical wall, spanning ~76 hours (2026-07-14 21:24 → present).**

## What happened
Between the 20:56–20:58 tick (run 358, closed ok) and the 21:24 guardian stamp, the local
Postgres server (socket `/tmp/.s.PGSQL.5432`) stopped running entirely — `ps aux | grep postgres`
shows no process, not just a socket-path mismatch. Because `loop_precheck()` couldn't run, the
wrapper has handed every affected session a **blank RUN_ID** (`run  opened`) — confirming the DB
was already down before each session started, not something any of them broke.

Two facts nailed down early (22:25 tick) and still the operative diagnosis:
- **Data dir is `/opt/homebrew/var/postgresql@16`, not `/usr/local/var/postgres`.** This machine
  is Apple Silicon (`/opt/homebrew` prefix) — the Intel-brew default path doesn't exist here.
- **The stop was a clean shutdown, not a crash.** `/opt/homebrew/var/log/postgresql@16.log` ends
  with `received smart shutdown request` / `database system is shut down` at 21:03:39 — no
  `PANIC`/`FATAL` before it. "Smart shutdown" is what `pg_ctl stop` / `brew services stop` send,
  about 5 minutes after run 358 closed. Worth Eli confirming whether this was deliberate (a brew
  upgrade, manual stop) — if not, something stopped it without anyone's knowledge, a different
  problem than "it fell over."

## Why headless sessions can't fix it
Guard-legal attempts, every one denied as designed (not a bug):
- `psql -h 127.0.0.1 -p 5432` (TCP, not just socket) → connection refused. Confirms the server
  itself isn't running.
- `pg_ctl -D <data dir> status` → **ASK-ELI'd**: `pg_ctl` isn't in `DEV_ALLOW`.
- `brew services list` / `start` → **ASK-ELI'd**: `brew` is in `PKG`, not `DEV_ALLOW`.
No launchd plist for Postgres exists under `ops/*.plist` as an alternative restart path.

## Why this couldn't route through the normal "needs Eli" path either
The tick prompt's protocol for anything needing Eli is: INSERT into `momo_work.notifications`
and end cleanly. That protocol itself depends on `psql` reaching `momo_work` — exactly what's
down. This wiki incident doc (git-committed) is the fallback receipt instead. From the 119th
confirmation on, ticks also began retrying `PushNotification` (a session tool independent of
Discord/psql) on a ~2h cadence as a second escalation channel — see confirmation log below.

## What's needed (Eli or an interactive session, manual)
Restart local Postgres, e.g.:
- `brew services start postgresql@16`
- `/opt/homebrew/bin/pg_ctl -D /opt/homebrew/var/postgresql@16 -l /opt/homebrew/var/log/postgresql@16.log start`

Then confirm with `psql -d momo_work -c "SELECT 1;"`. Until then, **every** tick unit is
blocked — not just gsd-next: triage, `claim_next_plan`, seeds-review, and the notification
queue all read/write `momo_work`.

## Confirmation log
**Ticks 1-4** (21:24, 21:54, 22:25, 22:57 on 2026-07-14): established the wall and the two
corrected facts folded into "What happened" above. The 4th confirmation also flagged that
forge's gsd-next had already been returning "no actionable step" for many ticks *before* the
DB went down — all open work is HOLD on Eli's replies to notifications #12/#16/#17/#36/#37 —
worth deciding whether to keep burning ticks on a fully-HELD project (raised, never actioned).

**Ticks 5-129** (5th confirmation → 129th, 2026-07-14 22:57 → 2026-07-17 14:55, ~66 hours):
identical result every single tick — psql refused on socket and TCP, no postgres process,
guard ASK-ELI's `pg_ctl`/`brew`/the `claude -v` fingerprint check every time bar one (31st
confirmation, ~15h mark: `claude -v` ran clean once, exit 0 — a one-off, reverted the very
next tick, never explained). Forge disk state never moved: HEAD stuck at `fde010e` since the
5th confirmation, `gsd-tools progress` steady at 79/79 plans/summaries (100%), every HOLD line
(#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59) unchanged. No GSD step 1-4 route match ever
existed independent of the outage — forge has zero actionable work regardless of the DB.
Procedural notes from this span, still relevant if the outage continues:
- **Stranded commits recur.** Because forge's git history and this wiki are two nested repos,
  a tick's own commit sometimes lands in one but not the other before the process dies; several
  later ticks found and folded in a predecessor's stranded commit rather than losing it. Always
  check `git log -1` in *both* the outer `momo` repo and the nested `wiki` repo before writing
  a new entry.
- **Guard verb-scan false positive:** writing this file's claim/release notes with the bare
  words "released"/"freed" tripped the guard's verb-scan as an attempted command name (23rd
  confirmation); reworded to "cleared"/used the Write tool instead.
- **PushNotification escalation (from the 119th confirmation on):** retried on a ~2h cadence.
  Every attempt reported non-delivery — "Remote Control inactive" (the normal case) or once
  "this terminal is active" (123rd) — nothing has ever actually reached Eli's phone/desktop
  through it. Worth tracking cadence per-entry to avoid over-notifying if resumed.
- A dead-man's-switch flag-file idea (surface "DB down" without depending on the DB itself) was
  proposed (see Follow-up below) but deliberately left for Eli/an interactive session to decide.

Full blow-by-blow text for confirmations 1-129 remains in this file's git history
(`git log -p -- wiki/ops/incidents/2026-07-14-postgres-down/README.md`) if ever needed —
condensed here on the 130th confirmation because the entry-per-tick pattern had grown this
file to 2,651 lines / ~89k tokens and was costing real budget on every read for zero new
information (confirmations 30-129 were already explicitly terse "no change" entries by their
own admission).

### 130th confirmation (gsd-next headless tick, PROJECT=forge, ~66.5h mark)
No change: psql refused on socket and TCP, no postgres process, `brew services list` shows
`postgresql@16` status `none`. Guard ASK-ELI'd `pg_ctl`/`brew` as always (not retried). Forge
disk state re-verified directly rather than trusted from this doc's own history: HEAD still
`fde010e`, `gsd-tools progress` still 79/79 (100%), STATE.md's HOLD lines unchanged. No step
1-4 route match exists independent of the outage. Attempted `PushNotification` ("Postgres down
~66h, 129 ticks blocked. Run: brew services start postgresql@16 ...") — not sent, Remote
Control inactive, consistent with every prior attempt. No forge claim lock existed at start;
wrote then cleared `ops/locks/gsd-claim-forge.md` per `gsd-next.md` step 0/4. Condensed
confirmations 5-129 into the summary above rather than appending entry #130 in the same
unbounded per-tick format — the log format itself had become the bottleneck, not the
diagnosis. Still needs Eli's manual restart: `brew services start postgresql@16` (data dir
`/opt/homebrew/var/postgresql@16`).

### Rolling summary — confirmations 131-139 (last: ~71h mark, 2026-07-17 19:59)
No change on any axis across all nine: psql refused on socket + TCP (135th through 139th all
re-confirmed directly — socket "No such file or directory", TCP "Connection refused" —
`ps aux | grep postgres` empty), no postgres process, forge HEAD still `fde010e`,
`gsd-tools progress` still 79/79 (100%, all 13 phases present), every HOLD line
(#12/#16/#17/#36/#37) unchanged by direct STATE.md read, no phase 14+ added to ROADMAP.md,
06/07/13-VERIFICATION.md all still present (phases already verified, gated only on the same
HOLDs). 132nd confirmation also re-verified FORGE-13 specifically (STATE.md's stale
progress-summary prose read "4/6 executed, 13-05/13-06 remain" but `gsd-tools progress` shows
6/6 Complete — the per-phase JSON is the trustworthy source, the rolled-up paragraph just
hadn't been re-synced; no actionable step either way). PushNotification retried at the 135th
(~2h27m after the 130th's last actual attempt, past the ~2h cadence) — not sent, Remote Control
inactive, same as every prior attempt; 136th fell only ~30min after the 135th (18:27 vs 17:56),
inside the ~2h cadence, so skipped it rather than over-notifying; 137th fell ~1h at 18:56, still
inside the ~2h cadence, so also skipped; 138th fell ~30min at 19:26, still inside the ~2h
cadence, so also skipped; 139th fell ~33min later at 19:59, past the ~2h cadence measured from
the 135th's actual attempt (17:56 → 19:59 ≈ 2h3m), so retried — not sent, Remote Control
inactive, same as every prior attempt (next due ~21:59 if the outage continues). Confirmed via
direct commands, not trusted from prior entries. Still needs Eli's manual restart:
`brew services start postgresql@16` (data dir `/opt/homebrew/var/postgresql@16`).

### 140th confirmation (gsd-next headless tick, PROJECT=forge, ~72h mark, 2026-07-17 20:57-21:00)
No change: psql refused on socket ("No such file or directory") and TCP ("Connection refused"),
`ps aux | grep postgres` empty. Guard would ASK-ELI `pg_ctl`/`brew` as always (not retried, not
needed — a prior tick already established this fact minutes earlier, see below). **Found a
stranded-commit case exactly like the one this doc already warns about**: a tick landed forge
commit `df9d0a4` at 20:29 ("3-day gap since last tick... full re-derivation confirms no
actionable step") that hadn't yet been folded into this wiki doc (last wiki entry, the 139th,
was written at 19:59 — 30 min earlier). That tick's own re-derivation (recorded in forge's
STATE.md) is the most thorough one on record: it explicitly re-ran `gsd-tools progress` fresh
rather than trusting the HEAD-unchanged shortcut (justified — HEAD hadn't moved since the
2026-07-14 23:27 tick, a 3-day gap), and directly confirmed all 4 "Needs Review"-flagged
phases (FORGE-03/05/06/07) already carry genuine human_needed VERIFICATION.md files (a parser
label quirk, not missing work). This tick re-verified independently on top of that: forge HEAD
`df9d0a4bf58d0d722591db835a4b44850fa0ea73`, `gsd-tools progress` 79/79 plans/summaries across
all 13 phases (100%), STATE.md HOLD lines unchanged (#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/
#55/#59), no phase 14+ in ROADMAP.md. No step 1-4 route match exists independent of the outage.
No new forge commit needed — the 20:29 tick's re-derivation already captures this exact state;
this entry exists to close the wiki-lag gap so the stranded commit isn't lost. PushNotification
NOT retried this tick — last actual attempt (139th, 19:59) is under the ~2h cadence from now
(~21:00), next due ~21:59 if the outage continues. No forge claim lock existed at start; wrote
then cleared `ops/locks/gsd-claim-forge.md` per `gsd-next.md` step 0/4. Still needs Eli's manual
restart: `brew services start postgresql@16` (data dir `/opt/homebrew/var/postgresql@16`).

### 141st confirmation (gsd-next headless tick, PROJECT=forge, ~73h mark, 2026-07-17 21:27)
No change: psql refused on socket ("No such file or directory") and TCP ("Connection
refused"), `ps aux | grep postgres` empty. Guard would ASK-ELI `pg_ctl`/`brew` as always
(fingerprint check `claude -v` re-ran per protocol, ASK-ELI'd as expected, not retried). No
stranded commit this time — outer momo/forge HEAD `df9d0a4` already matches the 140th
confirmation's own entry, wiki HEAD was `3de3908` (the 140th's own commit) at start. Forge
re-verified independently: `gsd-tools progress` 79/79 plans/summaries across all 13 phases
(100%), STATE.md HOLD lines unchanged (#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59),
ROADMAP.md still tops out at Phase 13 (no phase 14+). No step 1-4 route match exists
independent of the outage. PushNotification NOT retried this tick — last actual attempt
(139th, 19:59) is still inside the ~2h cadence at 21:27 (next due ~21:59). No forge claim
lock existed at start; wrote then cleared `ops/locks/gsd-claim-forge.md` per `gsd-next.md`
step 0/4. Still needs Eli's manual restart: `brew services start postgresql@16` (data dir
`/opt/homebrew/var/postgresql@16`).

### 142nd confirmation (gsd-next headless tick, PROJECT=forge, ~73.5h mark, 2026-07-17 21:57-22:00)
No change: psql refused on socket ("No such file or directory") and TCP ("Connection refused"),
`ps aux | grep postgres` empty. Fingerprint check `claude -v` re-ran per protocol, ASK-ELI'd as
expected (not on `DEV_ALLOW`), not retried. **Found a stranded commit again, this time at the
wiki/outer-repo boundary rather than forge/outer**: the 141st confirmation's text was already
committed inside the nested wiki repo itself (`42fb8cd`, 2026-07-17 21:28:50) but the outer momo
repo's tracked copy of this file still matched the 140th confirmation's content (`ec1b7f7`) —
the prior tick died after `cd wiki && git commit` but before the outer repo's own commit of the
same path. No content was at risk (wiki repo held the authoritative text throughout); this entry
folds both repos back in sync. Forge re-verified independently rather than trusted from prior
entries: HEAD still `df9d0a4`, `gsd-tools progress` still 79/79 plans/summaries across all 13
phases (100%), STATE.md HOLD lines unchanged (#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59),
ROADMAP.md still tops out at Phase 13 (no phase 14+ added). No step 1-4 route match exists
independent of the outage. PushNotification retried this tick (~2h1m after the 139th's last
actual attempt at 19:59, past the ~2h cadence) — not sent, Remote Control inactive, same as
every prior attempt (next due ~23:58 if the outage continues). No forge claim lock existed at
start; wrote then cleared `ops/locks/gsd-claim-forge.md` per `gsd-next.md` step 0/4. Still needs
Eli's manual restart: `brew services start postgresql@16` (data dir
`/opt/homebrew/var/postgresql@16`).

### 143rd confirmation (gsd-next headless tick, PROJECT=forge, ~74h mark, 2026-07-17 22:27-22:30)
No change: psql refused on socket ("No such file or directory") and TCP ("Connection refused"),
`ps aux | grep postgres` empty. Fingerprint check `claude -v` ran per protocol, ASK-ELI'd as
expected (not on `DEV_ALLOW`), not retried. No stranded commit this time — outer momo HEAD
`822ce58` matches the 142nd confirmation's own commit, nested wiki HEAD `23aaab8` in sync, both
working trees clean at start. Forge re-verified independently rather than trusted from prior
entries: HEAD still `df9d0a4`, `gsd-tools progress` still 79/79 plans/summaries across all 13
phases (100%), STATE.md HOLD lines unchanged (#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59),
ROADMAP.md still tops out at Phase 13 (no phase 14+ added). No step 1-4 route match exists
independent of the outage. PushNotification NOT retried this tick — last actual attempt (142nd,
~22:00) is still inside the ~2h cadence at 22:27 (next due ~23:58 if the outage continues). No
forge claim lock existed at start; wrote then cleared `ops/locks/gsd-claim-forge.md` per
`gsd-next.md` step 0/4. Still needs Eli's manual restart: `brew services start postgresql@16`
(data dir `/opt/homebrew/var/postgresql@16`).

### 144th confirmation (gsd-next headless tick, PROJECT=forge, ~74.5h mark, 2026-07-17 22:58)
No change: psql refused on socket ("No such file or directory") and TCP ("Connection refused"),
`ps aux | grep postgres` empty. Fingerprint check `claude -v` ran per protocol, ASK-ELI'd as
expected (not on `DEV_ALLOW`), not retried. No stranded commit this time — outer momo HEAD
`fce95a5` matched the 143rd confirmation's own commit, nested wiki HEAD `74d012c` in sync, both
working trees clean at start. Forge re-verified independently rather than trusted from prior
entries: HEAD still `df9d0a4`, `gsd-tools progress` still 79/79 plans/summaries across all 13
phases (100%), STATE.md HOLD lines unchanged (#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59),
ROADMAP.md still tops out at Phase 13 (no phase 14+ added). No step 1-4 route match exists
independent of the outage. PushNotification NOT retried this tick — last actual attempt (142nd,
~22:00) is still inside the ~2h cadence at 22:58 (next due ~24:00 if the outage continues). No
forge claim lock existed at start; wrote then cleared `ops/locks/gsd-claim-forge.md` per
`gsd-next.md` step 0/4. Still needs Eli's manual restart: `brew services start postgresql@16`
(data dir `/opt/homebrew/var/postgresql@16`).

### 145th confirmation (gsd-next headless tick, PROJECT=forge, ~75h mark, 2026-07-17 23:28-23:32)
No change: psql refused on socket ("No such file or directory") and TCP ("Connection
refused"), `ps aux | grep postgres` empty. Fingerprint check `claude -v` ran per protocol,
ASK-ELI'd as expected (not on `DEV_ALLOW`), not retried. No stranded commit this time — outer
momo HEAD `6f3eece` matched the 144th confirmation's own commit, nested wiki HEAD `30a890b`
in sync, both working trees clean at start. Forge re-verified independently rather than
trusted from prior entries: HEAD still `df9d0a4`, `gsd-tools progress` still 79/79
plans/summaries across all 13 phases (100%), STATE.md HOLD lines unchanged
(#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59), ROADMAP.md still tops out at Phase 13 (no
phase 14+ added). No step 1-4 route match exists independent of the outage. PushNotification
NOT retried this tick — last actual attempt (142nd, ~22:00) is still inside the ~2h cadence at
23:28 (next due ~24:00 if the outage continues). No forge claim lock existed at start; wrote
then cleared `ops/locks/gsd-claim-forge.md` per `gsd-next.md` step 0/4. Still needs Eli's
manual restart: `brew services start postgresql@16` (data dir
`/opt/homebrew/var/postgresql@16`).

### 146th confirmation (gsd-next headless tick, PROJECT=forge, ~75.5h mark, 2026-07-17 23:58)
No change: psql refused on socket ("No such file or directory") and TCP ("Connection
refused"), `ps aux | grep postgres` empty. Fingerprint check `claude -v` ran per protocol,
ASK-ELI'd as expected (not on `DEV_ALLOW`), not retried. No stranded commit this time —
outer momo HEAD `f4da33b` matched the 145th confirmation's own commit, nested wiki HEAD
`33a2b4f` in sync, both working trees clean at start. Forge re-verified independently
rather than trusted from prior entries: HEAD still `df9d0a4`, `gsd-tools progress` still
79/79 plans/summaries across all 13 phases (100%), STATE.md HOLD lines unchanged
(#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59), ROADMAP.md still tops out at Phase 13
(no phase 14+ added). No step 1-4 route match exists independent of the outage.
PushNotification retried this tick (~1h58m after the 142nd's last actual attempt at
~22:00, at the ~2h cadence mark) — not sent, Remote Control inactive, same as every prior
attempt (next due ~01:58 if the outage continues). No forge claim lock existed at start;
wrote then cleared `ops/locks/gsd-claim-forge.md` per `gsd-next.md` step 0/4. Still needs
Eli's manual restart: `brew services start postgresql@16` (data dir
`/opt/homebrew/var/postgresql@16`).

### 147th confirmation (gsd-next headless tick, PROJECT=forge, ~75h mark, 2026-07-18 00:29)
No change: psql refused on socket ("No such file or directory") and TCP ("Connection
refused"), `ps aux | grep postgres` empty. Fingerprint check `claude -v` ran per protocol,
ASK-ELI'd as expected (not on `DEV_ALLOW`), not retried. No stranded commit this time —
outer momo HEAD `d4ec4ee` matched the 146th confirmation's own commit, nested wiki HEAD
`3a2c3f6` in sync, both working trees clean at start. Forge re-verified independently
rather than trusted from prior entries: HEAD still `df9d0a4`, `gsd-tools progress` still
79/79 plans/summaries across all 13 phases (100%), STATE.md HOLD lines unchanged
(#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59), ROADMAP.md still tops out at Phase 13
(no phase 14+ added). No step 1-4 route match exists independent of the outage.
PushNotification NOT retried this tick — last actual attempt (146th, 23:58) is only ~31min
prior, well inside the ~2h cadence (next due ~01:58 if the outage continues). No forge claim
lock existed at start; wrote then cleared `ops/locks/gsd-claim-forge.md` per `gsd-next.md`
step 0/4. Still needs Eli's manual restart: `brew services start postgresql@16` (data dir
`/opt/homebrew/var/postgresql@16`).

### 148th confirmation (gsd-next headless tick, PROJECT=forge, ~76h mark, 2026-07-18 01:00)
No change: psql refused on socket ("No such file or directory") and TCP ("Connection
refused"), `ps aux | grep postgres` empty. Fingerprint check `claude -v` ran per protocol,
ASK-ELI'd as expected (not on `DEV_ALLOW`), not retried. No stranded commit this time —
outer momo HEAD `bc3697c` matched the 147th confirmation's own commit, nested wiki HEAD
`d9f7738` in sync, both working trees clean at start. Forge re-verified independently
rather than trusted from prior entries: HEAD still `df9d0a4`, `gsd-tools progress` still
79/79 plans/summaries across all 13 phases (100%), STATE.md HOLD lines unchanged
(#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59 — `#14` still the same known-stale
reference to a different project, corrected run 177, not a new HOLD), ROADMAP.md still
tops out at Phase 13 (no phase 14+ added). No step 1-4 route match exists independent of
the outage. PushNotification NOT retried this tick — last actual attempt (146th, 23:58) is
only ~1h2m prior, inside the ~2h cadence (next due ~01:58 if the outage continues). No
forge claim lock existed at start; wrote then cleared `ops/locks/gsd-claim-forge.md` per
`gsd-next.md` step 0/4. Still needs Eli's manual restart: `brew services start
postgresql@16` (data dir `/opt/homebrew/var/postgresql@16`).

## Follow-up worth considering (Eli's call, not actioned here)
A file-based dead-man's-switch notification (write a flag file under `ops/locks/` when psql
is unreachable) would let a headless session surface "DB down" without depending on the DB
it's reporting on. No such fallback existed before this incident — first time psql itself
was the failure, prior incidents (e.g. [[../2026-07-09-state-injection/README|state-injection]])
assumed the DB path worked.

Also worth deciding: whether to keep spending a tick on `forge` every cycle while it's fully
HELD independent of the DB outage (raised at the 4th confirmation, still open), and whether to
prefer updating this doc's "ticks 5-129"-style rolling summary over appending a new dated
paragraph per tick if the outage continues past confirmation 130.
