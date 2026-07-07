# Work-engine dashboard — persistence issue

## Symptom
The `:3100` work-engine dashboard (`/Users/momo/momo/dashboard`, `pnpm dev`) keeps dying between
sessions/turns. Root cause: MOMO can only launch it via the Bash `run_in_background` mechanism, and the
harness **reaps background tasks between turns** — so the dev server dies with them. Restarting it each time
is futile churn (it dies again the next turn).

## Why MOMO can't fix it alone
- `nohup`/`disown` to detach isn't on the guard's dev allowlist → guard asks.
- A **launchd agent** (the real fix — survives independently) lives in `~/Library/LaunchAgents/` which is
  OUTSIDE `/Users/momo/momo` → the guard blocks writes there. Needs Eli/sudo.

## The permanent fix (needs Eli, ~2 min, whenever dashboard uptime matters)
Create a launchd agent that keeps the dashboard alive + restarts it on crash, like the MOMO instance itself.
Eli runs (with sudo/his approval) something like a `~/Library/LaunchAgents/com.momo.dashboard.plist` with
`KeepAlive=true`, `WorkingDirectory=/Users/momo/momo/dashboard`, `ProgramArguments=[pnpm, dev]`, then
`launchctl load`. MOMO can draft the plist (in-repo), Eli installs it.

## Interim
MOMO restarts it on demand when Eli's using it. Don't auto-restart every turn — it just dies again.

## ⚠️ 2026-07-07: launchd agent installed but NOT ticking (unresolved)
Eli requested always-on. MOMO built `ops/dashboard-guardian.sh` (v1 nohup → FAILED: launchd reaps
timer-job children on exit; v2 tmux-detach) + `ops/com.momo.dashboard.plist` (RunAtLoad + StartInterval 60,
mirrors com.momo.agent). Eli ran `launchctl load` — `launchctl list` showed it loaded (exit 0), BUT the
guardian's stdout/err logs NEVER appear + the dashboard stays down → **the agent isn't executing the script
on the timer**. Likely cause: macOS 26 `launchctl load` is deprecated + may load-without-scheduling;
**`launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.momo.dashboard.plist` is the modern path**.
TODO (needs Eli, low priority — convenience view): unload + `bootstrap` the agent, then verify a tick runs
the tmux guardian. Until then MOMO restarts on demand. DON'T over-invest — FORGE is the priority.
