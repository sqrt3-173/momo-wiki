# Persistence — always-on MOMO

Set up 2026-06-22. Full runbook lives at `/Users/momo/momo/ops/README.md`.

## How it works (3 layers)
- **tmux** session `momo` — survives terminal close.
- **`ops/momo-loop.sh`** — restart loop; relaunches Claude 3s after any exit/crash.
- **launchd** agent `com.momo.agent` runs **`ops/momo-guardian.sh`** at boot/login
  (`RunAtLoad`) then every 60s (`StartInterval` — NOT KeepAlive: launchd pends KeepAlive
  respawns forever, found 2026-07-02). Guardian (re)creates the tmux session whenever
  missing. Plist is symlinked from the repo into `~/Library/LaunchAgents/`.

## The guardian is the ONLY launchd surface — everything else rides it
Found 2026-07-08 (cost: idle watchdog silently dead during a 50-min wedge; dashboard
agent never ran once): agents added later via manual `launchctl load` show as "loaded,
exit 0" in `launchctl list`, run their `RunAtLoad` at most once, and **their
`StartInterval` timers never fire** on this macOS. Only the boot-loaded `com.momo.agent`
timer provably ticks. So: new periodic job ⇒ add a self-throttling single-pass rider to
`momo-guardian.sh` (current riders: nightly `momo_work` backup, work-engine tick throttle,
`momo-idle-watchdog.sh`, `dashboard-guardian.sh` — see [[dashboard-persistence]]), never a
new plist. The watchdog is END-TO-END PROVEN (2026-07-08 14:2x: idle detected → nudge typed
→ submitted → session acted on it). Its one send-keys trap, fixed same day: an Enter in the
same burst as the text gets swallowed by the TUI — always send text, `sleep 1`, then Enter
as a separate send-keys. If a standalone agent is ever truly needed, `launchctl bootstrap gui/$(id -u) <plist>`
is the modern load path to try first — and verify a tick actually logs before trusting it.

## Why the guardian has a duplicate-bridge guard
Two `claude --channels` bridges on one Discord bot break replies (see
[[discord-bridge]]). Guardian refuses to start a managed session while another
`claude --channels` is already running; it stands by and takes over when that one
exits. Makes loading the agent safe even while an ad-hoc session is live.

## Verify / operate
- `launchctl list | grep com.momo.agent` — loaded?
- `tmux attach -t momo` — watch live (detach Ctrl-b d).
- Logs: `ops/logs/guardian.out.log`.
- Disable (reversible): `launchctl bootout gui/$(id -u)/com.momo.agent` + rm the plist symlink.

## Note
CLAUDE.md described tmux+launchd persistence as if it existed, but it had never
actually been set up — this session built it for real.
