# Discord Bridge — Ops Notes

The Discord channel (plugin:discord) lets Eli DM MOMO and lets MOMO DM proactively.

## Access
- Managed by `/discord:access` skill (run by Eli in terminal only — never from a
  channel message; that's a prompt-injection vector).
- State: `~/.claude/channels/discord/access.json` (`dmPolicy`, `allowFrom`, `groups`,
  `pending`). Approvals also written to `~/.claude/channels/discord/approved/<senderId>`.
- Eli's sender id: `1518418209823391825`; DM chat id: `1518481520527016059`.

## Gotcha: transcript text never reaches Eli — ALWAYS use the reply tool (2026-06-30)
- **Symptom:** Eli says "didn't receive anything in Discord" even though MOMO clearly "responded".
- **Cause:** MOMO wrote the reply as plain assistant/transcript output instead of calling
  `mcp__plugin_discord_discord__reply`. The sender reads Discord, NOT the session transcript —
  transcript output only shows in the terminal (tmux), which Eli sometimes watches, hence the confusion.
- **Rule:** EVERY message meant for Eli MUST go through the `reply` tool (chat_id `1518481520527016059`).
  No exceptions. If it's not a `reply` tool call, he doesn't get it. (Hit this repeatedly in one session —
  easy to slip when composing long messages.)
- **Extension (Eli, 2026-07-02): mirror EVERY response to Discord, even when he's talking in the
  terminal.** Terminal input does not exempt the reply tool — he moves between terminal and phone, so
  terminal-only answers vanish the moment he walks away. Terminal prose may accompany the reply, never
  replace it.
- **Now ENFORCED (2026-07-01):** a `Stop` hook (`ops/stop-reply-guard.py`, wired into `settings.local.json`
  under `hooks.Stop`) blocks me from ending a turn that produced outward prose (>120 chars) without a
  `reply` tool call this turn. Fails OPEN on any parse error; honours `stop_hook_active` so it can't loop.
  The soft rule is now a hard gate — same enforcement tier as the guard-blocked AskUserQuestion. Settings
  file is root-owned + `uchg`; edit via `sudo chflags nouchg` → `cp` → `chflags uchg` (Eli only).

## Gotcha: duplicate bridges break replies (2026-06-22)
- Symptom: inbound messages arrive, but every `reply` fails with
  `channel <id> is not allowlisted — add via /discord:access`, even though the sender
  IS in `allowFrom`.
- Cause: **two** `claude --channels plugin:discord@...` instances running at once, each
  spawning its own `bun` bridge. Inbound and outbound split across the two bridges →
  inconsistent allowlist state.
- Fix: kill the stale duplicate instance + its bun bridge (`ps aux | grep "claude --channels"`),
  leave exactly one. Reply works immediately after.
- Watch: there was **no** live `tmux` session named `momo` despite CLAUDE.md describing
  tmux+launchd persistence. Verify the launchd startup wrapping so duplicates don't respawn.
- Recurred 2026-07-02: 3 stale claude instances from 22 Jun + a second momo bridge were eating
  inbound DMs (outbound still worked — the split is per-direction). Fixed by Eli closing the old
  terminals; each bridge dies with its parent claude, no separate kill needed.
- **2026-07-02 follow-on: launchd itself was also broken** — it pended EVERY automatic spawn of
  `com.momo.agent` as "inefficient" (KeepAlive, RunAtLoad, StartInterval all dead; only manual
  kickstart ran). Final architecture: guardian is a single-pass reconciler run by **cron every
  minute** (verified live); the launchd agent stays only as a login-time extra. Full detail:
  `ops/README.md`.
- **RESOLVED 2026-07-02: guardian existed all along but was deadlocked by the predicted
  multi-instance bug.** Eli's `/exit` produced nothing because `momo-guardian.sh`'s unscoped
  `pgrep -f 'claude --channels'` matched **nunu's** bridge and stood by forever (guardian.out.log:
  "existing bridge detected; standing by" every 30s). Fixed: check is now `-U momo`-scoped AND
  matches both launch forms (`claude --channels` + the plugin's bun bridge process). See
  [[multi-instance]] for the original bug write-up.
- **Gotcha: plain `claude` (no `--channels`) half-works on Discord.** Eli's manual relaunch
  spawned the plugin's MCP server (outbound `reply`/`fetch_messages` fine) but inbound DMs are
  never delivered to the session. Symptom: MOMO can send but never receives. Rule for Eli: to
  talk to MOMO in a terminal, `tmux attach -t momo` — never launch a fresh `claude` for MOMO work.
