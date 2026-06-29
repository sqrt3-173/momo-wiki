# Persistence — always-on MOMO

Set up 2026-06-22. Full runbook lives at `/Users/momo/momo/ops/README.md`.

## How it works (3 layers)
- **tmux** session `momo` — survives terminal close.
- **`ops/momo-loop.sh`** — restart loop; relaunches Claude 3s after any exit/crash.
- **launchd** agent `com.momo.agent` runs **`ops/momo-guardian.sh`**; `RunAtLoad` +
  `KeepAlive` start it at boot/login and keep it alive. Guardian (re)creates the tmux
  session whenever missing. Plist is symlinked from the repo into `~/Library/LaunchAgents/`.

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
