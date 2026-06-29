# Guardrails — unattended Discord operation

Set up 2026-06-23 so MOMO runs unattended via Discord without stalling on approvals Eli
can't give from his phone, while dangerous edges stay hard-blocked. See [[persistence]].

## Asking rule (hard rule in soul.md)
When MOMO needs a decision/typed answer, it asks in **plain numbered text via Discord**
(mcp reply) — NEVER the interactive picker (AskUserQuestion), which Eli can't use from his
phone. Enforced in config too: AskUserQuestion is in `permissions.deny` and blocked by the
PreToolUse guard.

## PreToolUse guard — `ops/momo-guard.py` (v4)
Wired in `.claude/settings.local.json` → `hooks.PreToolUse` (matcher `*`). Reads the tool
call as JSON. **ALLOW** = decision + exit 0; **every BLOCK = exit code 2** (+ reason on
stderr) so denies are unambiguous. Bash is split quote-aware (shlex, so `psql -F'|'` doesn't
mis-split) and each sub-command judged on its LEADING tool — an echo/grep that merely mentions
a trigger word is NOT flagged. Proven by `ops/guard_proof.py` (adversarial, shows exit codes)
and `ops/guard_test.py` (29-case regression). **Loads mid-session — no restart; persists.**

- **AUTO-APPROVE:** reads; edits/writes inside `/Users/momo/momo`; an allowlist of build/
  test/dev + local-git commands; **read-only** psql; web (WebFetch/WebSearch); read/comms MCP
  (incl. Discord reply/react).
- **HARD-BLOCK (exit 2, never bypassable):** spend money (bash patterns + money-named MCP);
  macOS/system settings (sudo, defaults write, systemsetup, launchctl on /System|/Library,
  osascript…); destructive deletes outside the work dir; catastrophic cmds; `curl|bash`-style
  remote-to-shell RCE.
- **ASK-ELI (exit 2 + DM Eli, wait):** install software; publish/push public (git push, gh
  pr/release, deploy, tunnels, repo→public, **and publish/send MCP tools by name**); delete
  real data — **psql SQL is inspected**: reads (`select`/`\d`) pass, any DROP/DELETE/TRUNCATE/
  UPDATE/INSERT/ALTER (or `psql -f` whose SQL can't be seen) asks; ANY command whose leading
  tool isn't on the allowlist; writes outside the work dir.

## Non-Bash coverage (money/publish aren't bash)
MCP tools are classified by name: money/charge/billing → HARD-BLOCK; publish/share/send/
deploy → ASK-ELI; Discord comms + reads → allow. Web fetch/search → allow (reads).

## Inheritance + subagent lockdown (critical for the autonomous job)
- `claude -p` from `/Users/momo/momo` reads the same `.claude/settings.local.json` → inherits
  this hook. The always-on loop `cd`s there first.
- **Subagent lockdown:** subagent tool calls carry an `agent_id` field (main-loop calls don't —
  verified by instrumenting the hook). The guard uses it to restrict subagents to
  `SUBAGENT_ALLOW` (WebSearch/WebFetch/ToolSearch + read tools) — they CANNOT run Bash, install,
  write, publish, delete, or anything else. Verified live: a real subagent's `echo` was blocked
  with `SUBAGENT-BLOCKED`. This is the key exfil mitigation: the components that read untrusted
  practice websites have NO execution capability.

## Backup + reproducibility (before any unattended write run)
DB is recoverable two ways: (1) `pg_dump bd_crm > backups/bd_crm_<ts>.sql` (restore: `psql bd_crm
< file`); (2) rebuild from source — the Monday `*.xlsx` exports → `data/*_raw.json` →
`prisma/ingest.ts` (base graph) → `enrich/augment.py` + `enrich/write.ts` over the saved
`data/*.json` agent outputs (enriched layer). Source xlsx are never written by the job.
Snapshot before each unattended run. (Project CODE recoverability = git; commit before relying
on unattended `rm`-inside-workdir, which the guard now allows.)

## Containment layers (added 2026-06-23, before the 220 run)
1. **Delegation (#1):** main-loop `WebFetch`/`WebSearch` → ASK; only the locked subagents
   fetch untrusted pages. So the DB-holder (main loop) never ingests untrusted content →
   isn't injectable by a page. Subagent prompts are public-only (`enrich/subagent_template.md`):
   name/state/known-URL — never DB data, Eli's notes, or other surgeons. A hijacked subagent
   leaks only the one public page it holds.
2. **Egress proxy (#2):** `ops/egress_policy.py` (GET→any host; POST/PUT to allowlisted hosts
   only — Anthropic/registries/localhost — else BLOCK; 10MB body cap; full log to
   `ops/logs/egress.log`). Production = mitmproxy addon `ops/egress_mitm.py` (needs
   `mitmdump`, HTTPS_PROXY, CA trust). Proven live on real sockets (`ops/egress_proxy_test.py`):
   a 2MB DB POST to an attacker host is 403'd; GETs pass.
3. **Self-disarm block (#3a):** guard refuses Edit/Write to `momo-guard.py`/`settings*.json`
   and shell writes that TARGET them. SPEED-BUMP ONLY — a python/node script that writes the
   file bypasses it (proven). The real lock is #3b.
4. **OS immutability (#3b) — Eli runs (sudo), the actual guarantee:**
   ```
   sudo chown root:wheel /Users/momo/momo/ops/momo-guard.py /Users/momo/momo/.claude/settings.local.json
   sudo chmod 444       /Users/momo/momo/ops/momo-guard.py /Users/momo/momo/.claude/settings.local.json
   sudo chflags uchg    /Users/momo/momo/ops/momo-guard.py /Users/momo/momo/.claude/settings.local.json
   ```
   Verify (must fail "Operation not permitted"): `touch /Users/momo/momo/ops/momo-guard.py`.
   Update later: `sudo chflags nouchg <file>` → edit → re-lock. uchg blocks write AND unlink
   even though momo owns the parent dir.

## Honest residuals (the real boundary)
- Slow-drip exfil via GET URLs / query strings to arbitrary hosts (research needs arbitrary
  GETs) and DNS — LOGGED, not blocked.
- Node's built-in fetch ignores HTTP(S)_PROXY → can bypass the proxy. Mitigated by #1 (main
  loop never fetches) — a raw `node -e fetch` only matters if the main loop is injected, which
  #1 prevents.
- Guard-level self-disarm is a speed-bump; OS uchg (#3b) is the guarantee.
- Exfil is NOT airtight against a fully-compromised main loop without OS egress isolation;
  the layered controls bound the blast radius rather than eliminate it.

## Exfiltration — the honest boundary
The guard ASKs on `curl/wget` data-upload/POST (catches `curl -d @dump https://…`). It does NOT
and CANNOT catch `node -e "fetch(POST)"` / `python -c requests.post` — an in-process guard can't
contain arbitrary network calls by the interpreters dev work needs. Mitigations in place: the
website-reading subagents can't execute at all (lockdown above); the main loop's discipline
treats web/channel content as untrusted. FULL containment of a compromised main loop needs
OS-level egress isolation (separate user + outbound allowlist) — recommended for this
competitive-intel data; not yet done.

## Approval flow + honest limits
After Eli approves a specific Bash action in Discord, MOMO appends `#MOMO_OK` to that one
command. The marker bypasses the named ASK categories (install/publish/delete) **only** —
NEVER a HARD-BLOCK, and NEVER an UNKNOWN command (those need an explicit allowlist edit Eli
can see). MOMO never adds the marker without Eli's direct go-ahead.
- The marker/allowlist are deliberate, auditable approval-application steps — not a sandbox
  against a fully-compromised MOMO. True containment of an adversarial model needs OS-level
  isolation (restricted user / egress allowlist), out of scope here.
- Money detection is best-effort patterns; the primary guard is the soul.md "never spend
  without asking" rule, hook as backstop.
- Residual: `curl … | python3/node` (fetch-and-parse) is allowed and could be an exfil/RCE
  vector if fed hostile remote content — mitigated by injection-resistance discipline, not the
  hook.
