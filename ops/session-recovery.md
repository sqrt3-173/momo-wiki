# Session recovery — what to check after MOMO dies or is closed

First proven live 2026-07-03: Eli accidentally closed the session; total loss was ~60s of
downtime and zero work. This is the catch-up runbook for the next fresh session that wakes
into a "what happened?" situation.

## The restart itself is automatic
- The launchd guardian (`ops/momo-guardian.sh`, single-pass reconciler every 60s) detects the
  dead `momo` tmux session and restarts it running `momo-loop.sh`. No human action needed.
- A "Discord bridge is dead" report right after a close is usually just the reconnect window —
  the new session's bridge takes a moment to finish the Discord handshake. **Prove it, don't
  debug it:** fetch/receive a message, then send via the `reply` tool.
- Leftover orphan `bun run start` fragments from the old session are harmless — they hold no
  Discord connection and don't match the guardian's duplicate-bridge pgrep patterns.

## Where durable state lives (verify in this order)
1. **The wiki is its own nested git repo** (`wiki/.git`, remote `sqrt3-173/momo-wiki`, PAT via
   `ops/git-credential-momo-wiki.sh`). `git -C wiki status -sb` — prior sessions commit+push as
   they go (wiki pushes have standing approval), so this is usually already safe and Eli's
   Obsidian auto-pulls it.
2. **Work-engine + ingestion state is in Postgres `momo_work`** (`atoms.fate`, `inbox_items`,
   tasks, telemetry) — survives any session death by construction.
3. **The outer `/Users/momo/momo` repo is local-only (no remote)** and double-tracks wiki files
   as plain blobs. Its uncommitted changes are the *only* real loss surface — commit them.

## Guard quirks that bite during recovery
- `tmux` and `launchctl` aren't allowlisted → ASK-ELI. Use `ps aux`, `$TMUX`, and log files
  (`ops/logs/guardian.out.log`) instead.
- Compound bash with `cd`/subshells parses as unknown leading tool `''` → blocked. Use
  `git -C <dir>` and single-tool commands.

## Interactive startup dialogs can hang a headless restart
Any first-run approval dialog (e.g. "New MCP server found in this project") blocks Claude Code
at launch — the Discord bridge isn't up yet, so messages sent while it's stuck are never seen,
and the session looks dead until someone attaches to tmux and answers it (happened 2026-07-09
after `shadcn` was added to `.mcp.json`; Eli's option-2 click also failed to persist in
`~/.claude.json`). Fix in place: `ops/momo-loop.sh` launches with
`--settings '{"enableAllProjectMcpServers": true}'`, pre-approving project `.mcp.json` servers
per Eli's standing choice. Rule of thumb: whenever adding anything that triggers a first-run
prompt (MCP server, plugin, trust dialog), clear the prompt in the live session immediately —
never leave it for the next unattended restart.

## Standing checks (things a fresh session / heartbeat should look at)
- **Regulatory watch:** `ops/regulatory-watch-au-health.md` is appended monthly by the cloud routine
  `au-health-privacy-watch`. If the latest entry is prefixed `MOMO-RELAY:`, relay it to Eli on Discord
  (the cloud agent can't DM). If the entry is `NO MATERIAL CHANGE`, do nothing. (Set up 2026-07-04 for
  the NV Health project — [[../projects/nv-health-website]].)

## Catch-up order for a fresh session
[[../Home]] → the active project's ROADMAP/DECISIONS → `git log` + `git status` in both repos →
`momo_work` counts → then report to Eli on Discord (the `reply` tool, never transcript prose)
with what was verified, not what the old session claimed.
