# NUNU setup blueprint — second MOMO instance + shared memory/reasoning architecture

End state: a second autonomous agent **NUNU** running as macOS user `nunu` on MOMO-mini,
fully independent (own Claude account, own Discord bot/channel, own project space), running in
parallel with MOMO and surviving even when the `momo` user is the foreground login.

**Architecture (FINALISED 2026-07-01) — full separation, nothing shared:**
NUNU serves **Eli's sister Yana** and her marketing job at **Frank Developments** — a different person,
different domain — and is **fully standalone**: its own wiki AND its own skills, developed independently.
- **MOMO (master):** `momo-wiki` (knowledge) + `agent-skills` (its own skills repo) — both under **`sqrt3-173`** (Eli's account). MOMO is master admin of everything shared/infra.
- **NUNU:** `nunu-wiki` (knowledge + its own marketing skills) under the **`yana-nunu`** GitHub account. NUNU owns only its own domain; **no access to `momo-wiki` or `agent-skills`** (privacy boundary — different person's data).
- **Nothing is shared** between them (earlier drafts imagined a shared skill library — dropped for clean separation). Same machinery cloned, fully separate brains.
- NUNU's marketing skills (edm-campaign, email-html-scaffold, brand-fidelity-check — drafted 2026-07-01) live in **`nunu-wiki`**, seeded by MOMO via a scoped token, then NUNU develops them further.

Legend: **[ELI]** = only Eli can/should do it (account creation, payment, logins, secrets, sudo,
`/discord:access`). **[MOMO]** = I do it (all file scaffolding, scripts, config, git, code).

---

## The architecture, one picture
```
MOMO-mini (M2, 24GB)
├── user `momo`   → MOMO  → Claude acct A · Discord bot A (#momo)  · CLAUDE_CONFIG_DIR=~/.claude
├── user `nunu`  → NUNU  → Claude acct B · Discord bot B (#nunu)  · its own ~/.claude
│       (Fast User Switching: both logged in; NUNU runs in its background session)
├── Postgres (localhost:5432)            ← shared by both users natively
├── momo-wiki (git, private)             ← MOMO's knowledge, ELI's. clone: /Users/momo/momo/wiki
│       Eli's MacBook: clone + Obsidian Git plugin (two-way sync)
├── nunu-wiki (git, private, SEPARATE)   ← NUNU's knowledge, SISTER's domain. clone under nunu
│       NOT shared with momo-wiki — different person/job, privacy boundary
└── agent-skills (git, SHARED)           ← domain-agnostic playbooks BOTH instances clone; + council skill
```

---

## PHASE 0 — MOMO's wiki (`momo-wiki`, PRIVATE to Eli)  ✅ LIVE (2026-06-30)
> This is ELI's knowledge only. It is **not** shared with NUNU. NUNU gets its own `nunu-wiki` (Phase 1f).
Remote: `https://github.com/sqrt3-173/momo-wiki.git` (private). Token in `/Users/momo/.momo-secrets/github-wiki-token`
(600), read at push time by `ops/git-credential-momo-wiki.sh` (never stored in git config). MacBook/Obsidian sync = remaining sub-step.
1. **[MOMO]** Make the wiki a local git repo (init, `.gitignore`, first commit). ✅ done 2026-06-29.
2. **[ELI]** Create a **private** GitHub repo (e.g. `momo-wiki`). Empty. ✅ done (`sqrt3-173/momo-wiki`).
3. **[ELI]** Create a **fine-grained Personal Access Token** scoped to *only* that repo: ✅ done (saved to locked file).
   - Repository access → Only select repositories → `momo-wiki`.
   - Permissions → **Contents: Read and write**, **Metadata: Read** (auto). Nothing else (no Administration → can't delete/rename).
   - Set an expiry (e.g. 90 days; renew when it lapses).
   - **Save the token to a local file**, e.g. `/Users/momo/.momo-secrets/github-wiki-token` (chmod 600). **Do not paste it in Discord.** Tell MOMO the path.
4. **[MOMO]** Add the GitHub repo as the remote on MOMO's wiki clone; configure the token; push the first commit. ✅ done 2026-06-30 (remote `origin`, branch `main`, credential helper wired).
5. **[ELI, MacBook]** Clone the repo, open the folder as an Obsidian vault, install the **Obsidian Git** community plugin → set auto-pull on open + auto-commit/push on a timer. Now MacBook ↔ mini sync both ways. ⬅️ NEXT.
6. **[ELI decision]** Standing auto-push approval for `momo-wiki` — ✅ GRANTED 2026-06-30.
   - **Interim mode (active):** MOMO appends `#MOMO_OK` to `git push` commands **from the wiki dir only** under Eli's standing say-so. Works today, no guard change. Blast radius is tiny — the fine-grained token is contents-only on this one private repo, so worst case is a bad markdown commit (revertable). This is MOMO's discretion honouring Eli's word, not guard-enforced.
   - **Durable option — ✅ APPLIED 2026-06-30 (enforced, not discretionary):** the guard now auto-approves `git push` when the command contains the wiki dir path (`/users/momo/momo/wiki`, lowercased — matches the `git -C /Users/momo/momo/wiki push …` form MOMO always uses); all other pushes still ASK. Verified by a no-marker push succeeding. NUNU's guard will need the same edit.
   - **How the guard was edited (procedure — root-owned + `uchg` immutable + self-disarm-protected, so ONLY Eli/sudo):** MOMO generated + verified the patched file (diff = exactly the 2-line exemption, valid Python), then Eli ran: `sudo chflags nouchg <guard>` (unlock) → `sudo cp <patched> <guard>` (apply) → `sudo chflags uchg <guard>` (re-lock). MOMO is hard-blocked from all of these (chflags/cp targeting the guard = self-disarm). **Gotcha:** plain `sudo cp` fails with "Operation not permitted" until `chflags nouchg` clears the immutable flag — that's the "sudo + unlock" the guard docs mean. Also: multi-line heredocs pasted via Discord get mangled — prefer generate-file-then-cp.
   - Scope is strict: standing approval is for the `momo-wiki` repo ONLY — never extended to any other repo or push target.

Alternative if Eli wants nothing on third-party servers: **Syncthing** (peer-to-peer) instead of GitHub — lose clean history/merge, gain zero-cloud. (Git still recommended.)

---

## PHASE 1 — NUNU: the second instance
### 1a. macOS user
1. **[ELI]** System Settings → Users & Groups → add a new user **`nunu`** (Administrator if it must manage its own launchd; Standard is fine for a LaunchAgent under itself). Set a password.
2. **[ELI]** Enable **Fast User Switching** (Control Center / Users & Groups) so both `momo` and `nunu` can be logged in at once. Log into `nunu` once to create its home + login keychain, then switch back to `momo`. NUNU keeps running in `nunu`'s background session while `momo` is foreground — this is what satisfies "runs even when signed into the MOMO user."

### 1b. Claude account (separate quota)
3. **[ELI]** In `nunu`'s session, install/authenticate **Claude Code** with a **second Claude account** (second subscription = its own usage quota). Its config lives in `nunu`'s own `~/.claude` (separate home = separate keychain = clean isolation; no `CLAUDE_CONFIG_DIR` juggling needed because it's a different user).

### 1c. Discord bot + channel (separate, or replies break)
> Hard constraint: two `claude --channels` on the **same bot** break replies (documented in [[discord-bridge]]). NUNU needs its OWN bot.
4. **[ELI]** Discord Developer Portal → create a **second bot application** (e.g. "NUNU") → get its token → invite it to your server. Make a channel **#nunu**.
5. **[ELI]** In `nunu`'s session, run **`/discord:access`** to pair the new bot/channel to NUNU's Claude. (MOMO never touches this — anti-hijack control.) Stash the bot token in a local file, not Discord.

### 1d. Project space
6. **[MOMO/ELI]** Decide NUNU's working dir (e.g. `/Users/nunu/nunu` or a shared `/Users/Shared/projects/...`). MOMO scaffolds it.

### 1e. Always-on persistence (the "keeps running" part)
7. **[MOMO]** Clone MOMO's persistence stack for `nunu`, parametrised:
   - `nunu-guardian.sh` (SESSION=`nunu`), `nunu-loop.sh` (`cd` NUNU's dir; `claude --channels …`), `com.nunu.agent.plist`.
   - **Guardian fix (required):** MOMO's guardian uses a *global* `pgrep -f 'claude --channels'` and stands by if **any** bridge runs — so NUNU's guardian would refuse to start while MOMO's bridge is up. Scope the check to NUNU's own session/bot (e.g. match the tmux session name or config home), not "any bridge."
   - **Recommended:** install as a **LaunchAgent under `nunu`** (mirrors MOMO; runs in nunu's Aqua session via Fast User Switching; keychain is unlocked because the user is logged in → Claude auth works cleanly).
   - **Reboot note:** macOS auto-login supports only ONE user. After a reboot, `nunu` must be logged in once (manually, or via a helper) for NUNU to resume. Mini rarely reboots, so acceptable.
   - **Alternative (true unattended, reboot-proof):** a **LaunchDaemon** (`UserName=nunu`, in `/Library/LaunchDaemons`, needs **[ELI] sudo** to install — MOMO is hard-blocked from sudo/system changes). Caveat: a daemon may not have the user's unlocked login keychain → Claude auth could fail; would need the script to unlock the keychain or store creds outside it. More fragile. Use only if reboot-survival-with-nobody-logged-in is a hard requirement.
8. **[ELI]** Approve flipping the always-on job live (installing a 2nd persistent launchd job is a guard-ask). MOMO writes + shows everything first.

### 1f. NUNU's own wiki + shared skills (the brain + the methods)
9. **[ELI]** Create a **second private GitHub repo `nunu-wiki`** + its own fine-grained token (scoped to ONLY `nunu-wiki`, Contents r/w), saved to a locked file under `nunu`. This is NUNU's separate knowledge store for the sister's domain — never mixed with `momo-wiki`. **Ownership (revised 2026-07-01):** `nunu-wiki` lives under a **SEPARATE GitHub account for NUNU** (not Eli's main account) — cleaner isolation for the sister's domain. `agent-skills` (shared) can be owned by either account; the other instance reaches it via a repo-scoped token (fine-grained PATs are repo-scoped, not account-bound, so cross-account sharing just works).
10. **[MOMO]** Init NUNU's wiki clone under `nunu`, wire remote + credential helper (mirror of MOMO's setup), seed a Home/index for the sister's domain.
11. **[ELI]** Add the same narrow guard-edit to **NUNU's guard** so `git push` from NUNU's wiki dir auto-approves (root-owned + `uchg` → Eli/sudo; MOMO drafts the diff). See Phase 0 step 6 for the unlock→cp→re-lock procedure.
12. **[MOMO/ELI]** Both instances clone the **shared `agent-skills`** repo (see Phase 2a). NUNU reads the same playbooks; improvements propagate to both.

---

## NUNU bridge bring-up — AS-BUILT + gotchas (✅ working 2026-07-01)
NUNU is reachable on Discord end-to-end. Order that actually worked, and every trap hit:

1. **Discord bot** (separate Discord account for NUNU): Developer Portal → New Application "NUNU" → Bot → copy token → **enable MESSAGE CONTENT intent** → OAuth2 URL Generator (scope `bot`, perms Send Messages / Read Message History / View Channels) → invite to a **server you create** (bot must share a server with whoever DMs it).
2. **Token** → `/Users/nunu/.claude/channels/discord/.env` as `DISCORD_BOT_TOKEN=<token>` (chmod 600). (Same location/var as MOMO.)
3. **discord plugin** in NUNU's Claude: `/plugin marketplace add anthropics/claude-plugins-official` → `/plugin install discord@claude-plugins-official`.
4. **🚩 BIGGEST BLOCKER — org policy disables channels.** NUNU's account (Team org "Frank Developments") returned *"Channels are not enabled for your org — set channelsEnabled: true in managed settings."* MOMO's personal account has channels on by default (no managed-settings file existed). Fix (machine-wide, sudo): create `/Library/Application Support/ClaudeCode/managed-settings.json` = `{ "channelsEnabled": true }`. Harmless to MOMO. (If an org enforces it server-side, must toggle in the org admin console instead.)
5. **🚩 bun not installed for nunu.** The native Claude installer doesn't include bun, and MOMO's bun is home-scoped (`/Users/momo/.bun` — 700, unreachable). Install per-user: `curl -fsSL https://bun.sh/install | bash`.
6. **🚩 bun must be on PATH when Claude spawns the plugin's MCP server.** The discord plugin runs a `bun` server; a plain `claude --channels` launch didn't find bun (it's in the non-standard `~/.bun/bin`). Launch with it on PATH: `PATH="$HOME/.bun/bin:$PATH" claude --channels …`, or fallback `sudo ln -sf /Users/nunu/.bun/bin/bun /usr/local/bin/bun`. **The persistence loop (`nunu-loop.sh`) already bakes `/Users/nunu/.bun/bin` into PATH — so once deployed this can't recur.**
7. **Diagnosing "MCP failed":** `/mcp` → select discord shows status; `bun install` in the plugin dir confirmed deps were fine (115 pkgs); running the server by hand (`~/.bun/bin/bun run start`) printed `gateway connected as NUNU#5842` — proving the server works and the "error" stack trace was just a **harmless discord.js deprecation warning**.
8. **🚩 duplicate bridge.** A leftover manual `bun run start` (parent ≠ claude) is a second bot connection → breaks replies. Keep only the claude-spawned one (`bun run --cwd …/discord/… --silent start`, parent = the `claude --channels` pid). Kill leftovers by PID.
9. **Pairing / allowlist:** dmPolicy `pairing` → DM the bot → pairing code → `/discord:access pair <code>`. **`allowFrom` governs DMs only**; to talk in a server *channel*, `/discord:access group add <channelId>`.
10. **Verify from MOMO side** (can't read nunu's 700 home): `pgrep -u nunu -fl "bun run"` should show the discord bridge; `ps -o pid,ppid,command` to spot duplicates.

**🚩 Multi-account git keychain gotcha (2026-07-01):** on the mini, the macOS `osxkeychain` git helper caches ONE github.com credential (sqrt3-173's, from momo-wiki). Pushing to `yana-nunu/nunu-wiki` failed *"denied to sqrt3-173"* — keychain served the wrong account over the repo's token helper. Fix: force the repo to bypass keychain — `git config --add credential.helper ''` (empty resets the chain) then `--add credential.helper '!/…/git-credential-nunu-wiki.sh'`. Now the token helper is the only one. Same trap will hit any cross-account repo on this machine; NUNU's own user (fresh keychain) is clean, but if it ever caches, apply the same reset.

**✅ nunu-wiki SEEDED (2026-07-01):** MOMO seeded `yana-nunu/nunu-wiki` with NUNU's `soul.md`, `Home.md`, the 4 skills (edm-campaign, email-html-scaffold, brand-fidelity-check, community-engagement), and `projects/` (README + `_TEMPLATE` folder telling Yana exactly what to drop per project). Seed clone lives at `/Users/momo/momo/nunu-wiki-seed/` (MOMO's push copy; NUNU gets its own clone in its session).

**Still TODO for NUNU:** clone nunu-wiki into NUNU's session (`~/nunu/wiki`) + working dir; reply-tool permission / its own guard (fresh Claude has default perms); always-on persistence deploy (templates ready in `ops/nunu-templates/`). `agent-skills` (MOMO's own, `sqrt3-173`) — reorg MOMO's skills into it later.

## PHASE 2 — Memory + reasoning architecture
### 2a. Skill library (procedural memory — SHARED repo, build first, free)
- **[MOMO]** A standalone **`agent-skills`** git repo (NOT inside either wiki) — each repeatable methodology is a pull-from skill with a clear contract. Already seeded (currently living in `momo-wiki/skills/`): `website-audit`, `bd-research-sweep`, `entity-enrichment`, `council-review`, `wiki-capture`.
- **Shared by design:** both MOMO and NUNU clone it; the playbooks are domain-agnostic (how to research/audit/capture), so improvements help both. Only the *methods* are shared — the *knowledge* (wikis) stays separate.
- **[ELI]** When building NUNU: create a private `agent-skills` repo + token. **[MOMO]** then extracts `skills/` out of `momo-wiki` into `agent-skills`, both instances clone it, and `momo-wiki` keeps just a pointer. (Until then, skills live in `momo-wiki/skills/` and work fine for MOMO solo.)

### 2b. Wiki-capture protocol (semantic memory — free)
- **[MOMO]** A protocol/skill that fires at the end of every meaningful job: extract learnings → write linked, structured notes → update the index → commit/push. Plus a periodic **dedup/prune** pass.
- Discipline: capture-on-completion + prune, **not** an auto-generation firehose (unnavigable bloat = no memory). Curation over volume.

### 2c. Multi-model council (better reasoning — costs money, do last)
- **[ELI]** Fund + create API keys for the external models (OpenRouter = one key, many models: GPT-5, Gemini, etc.). MOMO is hard-blocked from spending — keys are Eli's.
- **[MOMO]** A `council` skill: for high-stakes calls (architecture, audits, ambiguous BD inferences), fan the question to Claude + 1–2 outside models, then synthesise + adversarially verify. Scope to hard calls only — overkill/wasteful on routine work.

---

## Build order (dependency-correct)
1. **Phase 0** `momo-wiki` — ✅ DONE 2026-06-30 (live, bidirectional mini↔MacBook, guard-enforced auto-push).
2. **Phase 2a/2b** skill library + wiki-capture — ✅ BUILT (currently in `momo-wiki/skills/`; extract to shared `agent-skills` when NUNU is built).
3. **Phase 1** NUNU instance — needs Eli: `nunu` user + 2nd Claude account + 2nd Discord bot + `nunu-wiki` repo + `agent-skills` repo.
4. **Phase 2c** council — last; needs Eli's funded OpenRouter key.

## Eli's immediate to-do to unblock (in parallel with MOMO's local work)
- [x] Private GitHub repo `momo-wiki` + fine-grained token → done; MacBook Obsidian sync live.
- [x] GitHub vs Syncthing → GitHub.
- [ ] **For NUNU:** create `nunu` user + enable Fast User Switching; set up 2nd Claude account; create the NUNU Discord bot + #nunu channel.
- [ ] **For NUNU's brain + shared methods:** create a **separate GitHub account for NUNU**; under it create private repo `nunu-wiki` + the shared `agent-skills` (each its own fine-grained token, Contents r/w) → tell MOMO the token file paths. (`agent-skills` owner = either account; both instances get a repo-scoped token.)
- [ ] Later: OpenRouter key for the council.
