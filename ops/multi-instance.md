# Running multiple MOMO instances (Option B) — architecture + plan

Eli wants 2+ separate MOMO instances so he can have multiple project builds going at once,
each steerable independently. Status (2026-06-28): **planned, not built** — waiting on Eli to
create the second Discord bot. Below is the worked-out architecture + the gotchas found by
reading the live setup.

## The core constraint: one Discord BOT per instance (not one channel)
Two `claude --channels plugin:discord@...` processes on the **same bot** break replies — inbound
and outbound split across two `bun` bridges, allowlist state goes inconsistent (documented in
[[discord-bridge]], hit 22 Jun). So each instance needs its **own bot**, not just its own channel.
The discord plugin config is global at `~/.claude/channels/discord/` (single bot), so isolation
requires each instance to run under its **own config home** (`CLAUDE_CONFIG_DIR` — verify exact
var at build time) holding its own paired bot + access.json.

## Per-instance ingredients
1. **Own Discord bot** (new application + token, invited to the server) — Eli creates in the
   Developer Portal. Token is a secret: stash in a local file, never paste in chat.
2. **Own channel** (e.g. #momo-clinic) paired via `/discord:access` — **Eli only**, in terminal
   (anti-hijack control; MOMO must never touch access.json or approve pairings).
3. **Own isolated config home** so it reads its own discord bot config (avoids the duplicate-bridge
   break).
4. **Own project working dir** under `projects/`.
5. **Own persistence stack** — clone `com.momo.agent.plist` → `com.momo2.agent`, guardian + loop with
   `SESSION=momo2`, `cd` to the new project, `--channels` pointing at its bot.

## Bug found in the current guardian (must fix before #2 works)
`ops/momo-guardian.sh` guards against duplicate bridges with a **global** check:
`pgrep -f 'claude --channels'` → it stands by if *any* bridge is running. With two instances, #2's
guardian sees #1's bridge and refuses to start. Fix: make the check **instance-scoped** (match its
own session/config home/bot), not "any bridge anywhere."

## Build split
- **Eli (can't be delegated):** create bot → new channel → `/discord:access` pair → stash token in a
  file → tell MOMO.
- **MOMO (after token + explicit go):** isolated config home; new project dir; clone+parametrise
  guardian/loop/plist as `com.momo2.agent`/`momo2`; fix the guardian's global-pgrep to instance-scoped;
  load the launchd agent. Installing the 2nd always-on launchd job is an ASK-Eli (persistent-system
  change) — write + show files first, flip live only on explicit OK.

## Hardware capacity (MOMO-mini)
M2 chip (8-core: 4P+4E), **24GB** unified RAM. Verdict (2026-06-29): comfortably runs **2 instances**
in parallel. RAM is not the bottleneck (2 instances + Postgres + ~dozen agents fits easily); CPU is the
tighter resource — research agents are I/O-bound (waiting on web fetches) so the 8 cores absorb more
concurrent-but-waiting tasks than 8. Pinch point = both instances firing max agent swarms (≥8 each)
simultaneously → contention = slower, never a crash. Guardrail: keep the ~6-agents/instance ceiling
(≈12 concurrent across two). Would only outgrow the box at 3+ heavy flat-out instances. Note: a separate
macOS user does NOT prevent parallelism — it's a permissions boundary, not a lock; macOS runs multiple
users' processes concurrently natively.

## What's shared vs separate (CORRECTED 2026-06-30)
NUNU serves Eli's **sister's job** — a different domain — so **knowledge is NOT shared**:
- **`momo-wiki`** (private, Eli's knowledge) and **`nunu-wiki`** (private, sister's domain) are **separate repos**. No mixing — privacy boundary.
- **`agent-skills`** (separate repo) IS **shared** — domain-agnostic how-to playbooks both instances clone; improvements propagate to both. Only the *methods* are shared, never the *knowledge*.
- **Database** — shared natively (both local users connect to the same Postgres `localhost:5432`; no dup).
- **Static files** both need → a shared local folder (`/Users/Shared/…`).
- **Why git not a raw shared folder for wikis/skills:** a raw shared folder = Dropbox-style clobber / no history / no merge. Git works even locally (bare repo, no internet); GitHub remote adds MacBook access + off-machine backup.
- **Repo security:** each repo gets its **own fine-grained PAT scoped to that one repo** (Contents r/w only): can't delete/rename, can't see other repos, instantly revocable. Eli owns repos; secrets stay out (.gitignore).
- **2nd-user always-on job** needs that user's session: keep logged in (fast user switching) OR LaunchDaemon (boots regardless of login) instead of a login-scoped LaunchAgent.

## Existing persistence stack (reference — instance #1)
launchd `com.momo.agent` (symlinked into `~/Library/LaunchAgents/`) runs `ops/momo-guardian.sh`
(KeepAlive) → creates tmux session `momo` running `ops/momo-loop.sh` → which runs
`claude --channels plugin:discord@claude-plugins-official` in a restart loop. See [[../../ops/README]].

## Alternative considered: better transport than Discord?
For multi-instance, Discord's channels ARE the right routing primitive — keep it. Only genuine upgrade
is a **custom Next.js web cockpit** (one pane per instance, all builds at once) — worth building at
~3+ instances; below that it's effort for little gain. Telegram = sidegrade. (Recommended to Eli
2026-06-28.)
