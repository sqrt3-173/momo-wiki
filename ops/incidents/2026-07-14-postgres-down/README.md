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

## Follow-up worth considering (Eli's call, not actioned here)
A file-based dead-man's-switch notification (write a flag file under `ops/locks/` when psql
is unreachable) would let a headless session surface "DB down" without depending on the DB
it's reporting on. No such fallback existed before this incident — first time psql itself
was the failure, prior incidents (e.g. [[../2026-07-09-state-injection/README|state-injection]])
assumed the DB path worked.
