# NUNU setup blueprint — second MOMO instance + shared memory/reasoning architecture

End state: a second autonomous agent **NUNU** running as macOS user `momo2` on MOMO-mini,
fully independent (own Claude account, own Discord bot/channel, own project space), running in
parallel with MOMO and surviving even when the `momo` user is the foreground login. Both share a
**git-backed wiki** (the master) and a **skill library**, and can call a **multi-model council**
for hard calls.

Legend: **[ELI]** = only Eli can/should do it (account creation, payment, logins, secrets, sudo,
`/discord:access`). **[MOMO]** = I do it (all file scaffolding, scripts, config, git, code).

---

## The architecture, one picture
```
MOMO-mini (M2, 24GB)
├── user `momo`   → MOMO  → Claude acct A · Discord bot A (#momo)  · CLAUDE_CONFIG_DIR=~/.claude
├── user `momo2`  → NUNU  → Claude acct B · Discord bot B (#nunu)  · its own ~/.claude
│       (Fast User Switching: both logged in; NUNU runs in its background session)
├── Postgres (localhost:5432)         ← shared by both users natively
├── Wiki = git repo                   ← master on GitHub (private); each instance has a clone
│       MOMO clone: /Users/momo/momo/wiki   NUNU clone: /Users/momo2/.../wiki
│       Eli's MacBook: clone + Obsidian Git plugin (two-way sync)
└── Skill library + council skill     ← shared (in the wiki repo or a shared git repo)
```

---

## PHASE 0 — Shared foundation: the git wiki master
1. **[MOMO]** Make the wiki a local git repo (init, `.gitignore`, first commit). *No pushing.* ✅ done 2026-06-29.
2. **[ELI]** Create a **private** GitHub repo (e.g. `momo-wiki`). Empty.
3. **[ELI]** Create a **fine-grained Personal Access Token** scoped to *only* that repo:
   - Repository access → Only select repositories → `momo-wiki`.
   - Permissions → **Contents: Read and write**, **Metadata: Read** (auto). Nothing else (no Administration → can't delete/rename).
   - Set an expiry (e.g. 90 days; renew when it lapses).
   - **Save the token to a local file**, e.g. `/Users/momo/.momo-secrets/github-wiki-token` (chmod 600). **Do not paste it in Discord.** Tell MOMO the path.
4. **[MOMO]** Add the GitHub repo as the remote on MOMO's wiki clone; configure the token; push the first commit. *(Push is guard-ask → I confirm with Eli before the first push.)*
5. **[ELI, MacBook]** Clone the repo, open the folder as an Obsidian vault, install the **Obsidian Git** community plugin → set auto-pull on open + auto-commit/push on a timer. Now MacBook ↔ mini sync both ways.

Alternative if Eli wants nothing on third-party servers: **Syncthing** (peer-to-peer) instead of GitHub — lose clean history/merge, gain zero-cloud. (Git still recommended.)

---

## PHASE 1 — NUNU: the second instance
### 1a. macOS user
1. **[ELI]** System Settings → Users & Groups → add a new user **`momo2`** (Administrator if it must manage its own launchd; Standard is fine for a LaunchAgent under itself). Set a password.
2. **[ELI]** Enable **Fast User Switching** (Control Center / Users & Groups) so both `momo` and `momo2` can be logged in at once. Log into `momo2` once to create its home + login keychain, then switch back to `momo`. NUNU keeps running in `momo2`'s background session while `momo` is foreground — this is what satisfies "runs even when signed into the MOMO user."

### 1b. Claude account (separate quota)
3. **[ELI]** In `momo2`'s session, install/authenticate **Claude Code** with a **second Claude account** (second subscription = its own usage quota). Its config lives in `momo2`'s own `~/.claude` (separate home = separate keychain = clean isolation; no `CLAUDE_CONFIG_DIR` juggling needed because it's a different user).

### 1c. Discord bot + channel (separate, or replies break)
> Hard constraint: two `claude --channels` on the **same bot** break replies (documented in [[discord-bridge]]). NUNU needs its OWN bot.
4. **[ELI]** Discord Developer Portal → create a **second bot application** (e.g. "NUNU") → get its token → invite it to your server. Make a channel **#nunu**.
5. **[ELI]** In `momo2`'s session, run **`/discord:access`** to pair the new bot/channel to NUNU's Claude. (MOMO never touches this — anti-hijack control.) Stash the bot token in a local file, not Discord.

### 1d. Project space
6. **[MOMO/ELI]** Decide NUNU's working dir (e.g. `/Users/momo2/nunu` or a shared `/Users/Shared/projects/...`). MOMO scaffolds it.

### 1e. Always-on persistence (the "keeps running" part)
7. **[MOMO]** Clone MOMO's persistence stack for `momo2`, parametrised:
   - `nunu-guardian.sh` (SESSION=`nunu`), `nunu-loop.sh` (`cd` NUNU's dir; `claude --channels …`), `com.nunu.agent.plist`.
   - **Guardian fix (required):** MOMO's guardian uses a *global* `pgrep -f 'claude --channels'` and stands by if **any** bridge runs — so NUNU's guardian would refuse to start while MOMO's bridge is up. Scope the check to NUNU's own session/bot (e.g. match the tmux session name or config home), not "any bridge."
   - **Recommended:** install as a **LaunchAgent under `momo2`** (mirrors MOMO; runs in momo2's Aqua session via Fast User Switching; keychain is unlocked because the user is logged in → Claude auth works cleanly).
   - **Reboot note:** macOS auto-login supports only ONE user. After a reboot, `momo2` must be logged in once (manually, or via a helper) for NUNU to resume. Mini rarely reboots, so acceptable.
   - **Alternative (true unattended, reboot-proof):** a **LaunchDaemon** (`UserName=momo2`, in `/Library/LaunchDaemons`, needs **[ELI] sudo** to install — MOMO is hard-blocked from sudo/system changes). Caveat: a daemon may not have the user's unlocked login keychain → Claude auth could fail; would need the script to unlock the keychain or store creds outside it. More fragile. Use only if reboot-survival-with-nobody-logged-in is a hard requirement.
8. **[ELI]** Approve flipping the always-on job live (installing a 2nd persistent launchd job is a guard-ask). MOMO writes + shows everything first.

---

## PHASE 2 — Memory + reasoning architecture (shared by MOMO + NUNU)
### 2a. Skill library (procedural memory — build first, free)
- **[MOMO]** A `skills/` library where each repeatable methodology is a pull-from skill with a clear contract. Seed from what we already run: `website-audit`, `bd-research-sweep`, `entity-enrichment`, `council-review`, `wiki-capture`. Version-controlled (in the shared git). Both instances pull from it.

### 2b. Wiki-capture protocol (semantic memory — free)
- **[MOMO]** A protocol/skill that fires at the end of every meaningful job: extract learnings → write linked, structured notes → update the index → commit/push. Plus a periodic **dedup/prune** pass.
- Discipline: capture-on-completion + prune, **not** an auto-generation firehose (unnavigable bloat = no memory). Curation over volume.

### 2c. Multi-model council (better reasoning — costs money, do last)
- **[ELI]** Fund + create API keys for the external models (OpenRouter = one key, many models: GPT-5, Gemini, etc.). MOMO is hard-blocked from spending — keys are Eli's.
- **[MOMO]** A `council` skill: for high-stakes calls (architecture, audits, ambiguous BD inferences), fan the question to Claude + 1–2 outside models, then synthesise + adversarially verify. Scope to hard calls only — overkill/wasteful on routine work.

---

## Build order (dependency-correct)
1. **Phase 0** git wiki (foundation everything shares) — MOMO local done; needs Eli's repo+token to go remote.
2. **Phase 2a/2b** skill library + wiki-capture — free, no dependencies, sharpens everything after. MOMO can build now.
3. **Phase 1** NUNU instance — needs Eli's macOS user + 2nd Claude account + 2nd Discord bot.
4. **Phase 2c** council — last; needs Eli's funded API keys.

## Eli's immediate to-do to unblock (in parallel with MOMO's local work)
- [ ] Private GitHub repo `momo-wiki` + fine-grained token (Phase 0.2–0.3) → tell MOMO the token file path.
- [ ] Decide GitHub vs Syncthing for the wiki.
- [ ] When ready for NUNU: create `momo2` user + enable Fast User Switching; set up 2nd Claude account; create the NUNU Discord bot + #nunu channel.
- [ ] Later: OpenRouter key for the council.
