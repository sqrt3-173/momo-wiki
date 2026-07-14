# Incident: local Postgres down — momo_work unreachable (2026-07-14)

**Severity:** high (blocks the whole tick engine, not just one unit). **Outcome:** open —
needs a manual restart only Eli or an interactive session can do; this headless tick could
not self-heal by design (guard correctly blocks the tools needed).

## What happened
Between the 20:56–20:58 tick (run 358, closed ok) and the 21:24 guardian stamp, the local
Postgres server (socket `/tmp/.s.PGSQL.5432`, data dir `/usr/local/var/postgres`) stopped
running entirely — `ps aux | grep postgres` shows no process, not just a socket-path mismatch.
`tick.log` already shows the raw signal:

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

## Why this couldn't route through the normal "needs Eli" path either
The tick prompt's protocol for anything needing Eli is: INSERT into `momo_work.notifications`
and end cleanly (headless sessions have no Discord tools). That protocol itself depends on
`psql` reaching `momo_work` — which is exactly what's down. No notification could be queued.
This incident note (git-committed, wiki repo) is the fallback receipt instead.

## What's needed (Eli or an interactive session, manual)
Restart local Postgres, e.g. one of:
- `brew services start postgresql@16` (binary found at `/opt/homebrew/opt/postgresql@16`)
- `/opt/homebrew/bin/pg_ctl -D /usr/local/var/postgres start`

Then confirm with `psql -d momo_work -c "SELECT 1;"`. Until then, **every** tick unit is
blocked — not just gsd-next: triage, `claim_next_plan`, seeds-review, and the notification
queue all read/write `momo_work`. Ticks will keep firing on schedule and keep failing the
same way (see `tick.log`), burning guardian stamps for nothing.

## Follow-up worth considering (Eli's call, not actioned here)
A file-based dead-man's-switch notification (write a flag file under `ops/locks/` when psql
is unreachable) would let a headless session surface "DB down" without depending on the DB
it's reporting on. No such fallback existed before this incident — first time psql itself
was the failure, prior incidents (e.g. [[../2026-07-09-state-injection/README|state-injection]])
assumed the DB path worked.
