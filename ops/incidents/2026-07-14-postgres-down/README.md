# Incident: local Postgres down — momo_work unreachable (2026-07-14)

**Severity:** high (blocks the whole tick engine, not just one unit). **Outcome:** open —
needs a manual restart only Eli or an interactive session can do; this headless tick could
not self-heal by design (guard correctly blocks the tools needed).

## What happened
Between the 20:56–20:58 tick (run 358, closed ok) and the 21:24 guardian stamp, the local
Postgres server (socket `/tmp/.s.PGSQL.5432`) stopped running entirely — `ps aux | grep postgres`
shows no process, not just a socket-path mismatch. `tick.log` already shows the raw signal:

```
[guardian] 2026-07-14 21:24:00 tick trigger=stamp
psql: error: connection to server on socket "/tmp/.s.PGSQL.5432" failed: No such file or directory
[tick] 2026-07-14 21:24:00 precheck paused= pending= new= plans= seeds= live= stale=
[tick] 2026-07-14 21:24:01 unit: gsd-next project=forge
psql: error: connection to server on socket "/tmp/.s.PGSQL.5432" failed: No such file or directory
[tick] 2026-07-14 21:24:01 run  opened
```

Because `loop_precheck()` couldn't run, the wrapper handed this session a **blank RUN_ID**
(`run  opened`, `ops/locks/run-.pid` contains just a trailing char) — confirming the DB was
already down before this session started, not something this session broke.

## Why this session couldn't fix it
Tried, in order, all guard-legal:
- `psql -h 127.0.0.1 -p 5432` (TCP, not just socket) → connection refused. Confirms the server
  itself isn't running, not a socket-path config issue.
- `pg_ctl -D /usr/local/var/postgres status` → **guard ASK-ELI'd it**: `pg_ctl` isn't in
  `DEV_ALLOW` (`ops/momo-guard.py`).
- `brew services list` → **guard ASK-ELI'd it**: `brew` is in `PKG`, not `DEV_ALLOW`.

Both denials are the guard working as designed (§ "any command whose leading tool isn't on
the allowlist" → ASK-ELI), not a bug. No launchd plist for Postgres exists under `ops/*.plist`
to check as an alternative restart path.

### Correction (22:25 tick, run blank — 3rd consecutive tick to hit this)
Same failure confirmed again, no change. Two corrections to the record above, found by reading
files directly (`ls`, `cat`/`grep` on config — no new blocked commands attempted, `pg_ctl`/`brew`
were NOT retried):
- **Data dir is `/opt/homebrew/var/postgresql@16`, not `/usr/local/var/postgres`.** This machine
  is Apple Silicon (`/opt/homebrew` homebrew prefix) — `/usr/local/var/postgres` is the Intel-brew
  default path and doesn't exist here (`ls` confirms). The real data dir has a live `PG_VERSION`
  + `base/`. The restart command below is corrected accordingly.
- **The stop was a clean shutdown, not a crash.** `/opt/homebrew/var/log/postgresql@16.log` ends:
  ```
  2026-07-14 21:03:39.892 AEST [5554] LOG:  received smart shutdown request
  2026-07-14 21:03:40.920 AEST [5554] LOG:  database system is shut down
  ```
  "smart shutdown" is what `pg_ctl stop` / `brew services stop` send — something asked Postgres
  to stop gracefully at 21:03:39, about 5 minutes after run 358 closed (20:56–20:58). No crash
  signature (no `PANIC`/`FATAL` before the shutdown line). Worth Eli confirming whether this was
  deliberate (a brew upgrade, manual stop) — if not, something on this machine stopped it without
  anyone's knowledge, which is a different problem than "it fell over."

## Why this couldn't route through the normal "needs Eli" path either
The tick prompt's protocol for anything needing Eli is: INSERT into `momo_work.notifications`
and end cleanly (headless sessions have no Discord tools). That protocol itself depends on
`psql` reaching `momo_work` — which is exactly what's down. No notification could be queued.
This incident note (git-committed, wiki repo) is the fallback receipt instead.

## What's needed (Eli or an interactive session, manual)
Restart local Postgres, e.g. one of:
- `brew services start postgresql@16`
- `/opt/homebrew/bin/pg_ctl -D /opt/homebrew/var/postgresql@16 -l /opt/homebrew/var/log/postgresql@16.log start`
  (corrected data dir — see 22:25 correction above; `/usr/local/var/postgres` does not exist
  on this Apple Silicon machine)

Then confirm with `psql -d momo_work -c "SELECT 1;"`. Until then, **every** tick unit is
blocked — not just gsd-next: triage, `claim_next_plan`, seeds-review, and the notification
queue all read/write `momo_work`. Ticks will keep firing on schedule and keep failing the
same way (see `tick.log`), burning guardian stamps for nothing. **3 ticks have now hit this
identical wall (21:24, 21:54, 22:25) at ~$0.6-1.4 equivalent usage each** — cheap individually
but this won't self-resolve; it needs one manual restart.

### 4th confirmation (22:57 tick, run blank)
Same failure, no change (`psql` still fails on the socket). Did NOT retry `pg_ctl`/`brew` —
both are already-established ASK-ELI blocks and retrying them burns budget for a result we
already have. Instead did the actual `gsd-next forge` unit disk-side (that work is git/file
based, not DB based — only the receipt/notification layer needs Postgres): checked `git log`
first and found HEAD unchanged since the 21:30 tick's commit, so nothing on disk could have
moved; skipped re-running the full progress/VERIFICATION.md re-derivation since it would just
reproduce an identical result. Landed the resume-point update to forge's STATE.md as the
receipt (commit `b35cdb3`), same fallback pattern this incident doc itself uses.

**Worth flagging when Eli's next reachable:** this is the 4th tick in a row to hit the same
wall (21:24, 21:54, 22:25, 22:57) — roughly 1.5 hours with zero notification/control_commands
visibility. Separately, `forge`'s gsd-next has now returned "no actionable step" for a long
run of ticks even before the DB went down (all open work is HOLD on Eli's replies to
notifications #12/#16/#17/#36/#37) — the wrapper will keep spending a tick on `forge` every
cycle until either Eli answers one of those, or an interactive session reprioritizes the
wrapper's project scan. Worth deciding whether to keep burning ticks on a fully-HELD project.

### 5th confirmation (headless gsd-next tick, blank RUN_ID)
`psql -d momo_work -t -A -c "SELECT 1;"` still fails on the same socket
(`/tmp/.s.PGSQL.5432`, "No such file or directory"). Did not retry `pg_ctl`/`brew` — both
remain established ASK-ELI blocks per the guard, retrying burns budget for a known answer.
Followed the shortcut this incident doc's 4th entry established: checked `git log` in
`projects/forge` first — HEAD is still `b35cdb3` (the 4th confirmation's own commit), so
nothing on disk moved and a full progress/VERIFICATION.md re-derivation would reproduce
the identical result. Landed a resume-point note in forge's STATE.md as the receipt
(same fallback this incident doc itself uses) rather than repeating the expensive read.
**5 ticks have now hit this identical wall (21:24, 21:54, 22:25, 22:57, this one)** —
Eli still needs to do the manual restart below; nothing new to add to the diagnosis.

### 6th confirmation (gsd-next headless tick, blank RUN_ID, PROJECT=forge)
`psql -d momo_work -t -A -c "SELECT 1;"` still fails on the same socket. Did not retry
`pg_ctl`/`brew` directly — both are established ASK-ELI blocks; confirmed the guard still
treats `brew` that way this tick (`brew services list` → ASK-ELI, same denial as before).
Checked disk state independently rather than trusting this doc's own prior entries: forge's
`git log -1` is still `fde010e` (the 5th confirmation's own commit, unchanged), and
`gsd-tools progress` reports 79/79 plans have summaries (100%) — consistent with STATE.md's
account that every phase is executed/verified and all that remains are HOLD device-checkpoints
and Eli-decision gates (notifications #12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59). No
actionable, non-held step exists for forge independent of the DB outage. Did not add a new
no-op commit to forge's own STATE.md this time (nothing changed there since the 5th
confirmation and forge is a real client repo — this wiki incident doc is the right place for
DB-outage receipts, not forge's git history). **6 ticks have now hit this identical wall.**
No notification could be queued (same root cause as every prior entry) and no code/plan
changes were made. Nothing new to add to the diagnosis or the manual-restart instructions
above — still needs Eli or an interactive session to run one of the two restart commands.

### 7th confirmation (gsd-next headless tick, blank RUN_ID, PROJECT=forge)
`psql -d momo_work -t -A -c "SELECT 1;"` still fails on the same socket, same error. Did not
retry `pg_ctl`/`brew` — both remain established ASK-ELI blocks; no new information to gain by
repeating them. Ran the fingerprint check (`claude -v`) as the tick prompt requires — guard
correctly ASK-ELI'd it (not on the dev allowlist), noted and moved on. Checked disk state
independently: forge's `git log -1` is still `fde010e` (unchanged since the 6th confirmation),
`gsd-tools progress` still reports 79/79 plans with summaries (100%), and STATE.md's account is
unchanged — every phase executed/verified, remaining items are all HOLD on Eli-decision or
device-checkpoint notifications (#12/#16/#17/#24/#30/#36/#37/#38/#47/#48/#55/#59). No actionable
non-held step exists independent of the DB outage. Did not add a no-op commit to forge's own
STATE.md (same reasoning as the 6th confirmation — nothing changed there, and forge is a real
client repo). No notification could be queued (same root cause as every prior entry). **7 ticks
have now hit this identical wall** (21:24, 21:54, 22:25, 22:57, 5th, 6th, this one) — still
needs Eli or an interactive session to run one of the two restart commands above. Nothing new
to add to the diagnosis.

### 8th confirmation (gsd-next headless tick, blank RUN_ID, PROJECT=forge)
`psql -d momo_work -t -A -c "SELECT 1;"` still fails on the same socket, same error. Did not
retry `pg_ctl`/`brew` — both remain established ASK-ELI blocks. Ran the fingerprint check
(`claude -v`) as required — guard correctly ASK-ELI'd it, noted and moved on. Checked disk
state directly: forge's `git log -1` is still `fde010e` (unchanged since the 6th/7th
confirmations), `gsd-tools progress` still reports 79/79 plans with summaries (100%), and
`ROADMAP.md` shows no phase with an unsatisfied-dependency gap — every phase 1-13 either has
plans+summaries landed or is marked complete. STATE.md's `stopped_at` is unchanged: Phase 12
Wave B held on notification #37, Phase 13-06 Task 2 is a non-blocking device checkpoint. No
step 1-4 route match exists independent of the DB outage. Did not add a no-op commit to
forge's own STATE.md (nothing changed there; forge is a real client repo, this wiki doc is
the receipt). No notification could be queued (same root cause as every prior entry). **8
ticks have now hit this identical wall** (21:24, 21:54, 22:25, 22:57, 5th, 6th, 7th, this
one) — still needs Eli or an interactive session to run one of the two restart commands
above. Nothing new to add to the diagnosis.

## Follow-up worth considering (Eli's call, not actioned here)
A file-based dead-man's-switch notification (write a flag file under `ops/locks/` when psql
is unreachable) would let a headless session surface "DB down" without depending on the DB
it's reporting on. No such fallback existed before this incident — first time psql itself
was the failure, prior incidents (e.g. [[../2026-07-09-state-injection/README|state-injection]])
assumed the DB path worked.
