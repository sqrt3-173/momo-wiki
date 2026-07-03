# Phase 4 probe findings (plan 33) — 2026-07-03

One unscheduled `claude -p` launch with true cron parentage (cron → guardian → `ops/momo-probe-tick.sh`;
the orphaned probe re-parented to launchd after the guardian's single-pass exit, as expected).
Window: 13:18:01–13:18:25. Session: `a56e3acb-b1ae-4e34-b619-921e4f465157`, 5 turns, exit 0.
Raw receipts: `ops/logs/probe-out.json`, `ops/logs/probe.log`.

## Verdicts per research assumption

- **A1 — guard inheritance: PASS.** The fingerprint BLOCK line, verbatim from guard.log (13:18:16,
  inside the window; the executor was quiesced to Read-only polling, so only the probe session could
  produce it):
  `2026-07-03T13:18:16 BLOCK Bash :: ASK-ELI: 'claude' isn't on the dev allowlist. ...`
  Supporting context: 8 ALLOW lines in the window (psql read, git status, soul.md Read path).
  Double receipt: the JSON result itself carries `permission_denials` naming the `claude -v` Bash
  call — the harness recorded the guard denial structurally. The headless session also demonstrated
  IL-02 containment live (a tick session cannot spawn nested sessions).
- **A2 — no duplicate bridge under `--strict-mcp-config`: PASS.** Bridge PID sets identical
  PRE/DURING(×3)/POST: `channels:[74228] bun-bridge:[74254]`. No new bridge process ever appeared.
- **A3 — OAuth from cron context: PASS.** `is_error: false`, real result text, no auth error subtype.
- **A4 — tool availability: as designed.** `flock` ABSENT, `timeout` ABSENT (mkdir-lock + bash
  watchdog patterns confirmed necessary); `/usr/bin/python3` = 3.9.6.
- **A5 — JSON field names, pinned verbatim for the plan-35 parser:**
  top-level `type, subtype, is_error, api_error_status, duration_ms, num_turns, result, stop_reason,
  session_id, total_cost_usd, usage, modelUsage, permission_denials, terminal_reason, uuid`;
  `usage` subkeys: `input_tokens, cache_creation_input_tokens, cache_read_input_tokens, output_tokens`
  (plus server_tool_use/cache_creation/iterations metadata); `modelUsage` = per-model map with
  `inputTokens/outputTokens/cacheReadInputTokens/cacheCreationInputTokens/costUSD`.
  Parser rule: tokens = sum of the four `usage` `*_tokens` ints; cost = `total_cost_usd`.
- **A6 — `--max-budget-usd`: ACCEPTED** under subscription OAuth (flag taken, session ran).
- **A7 — CLI 2.1.199 (Claude Code).** All flags accepted: `--output-format json, --max-turns,
  --permission-mode dontAsk, --strict-mcp-config, --max-budget-usd`.

## Bonus finding — real cost data exists

`total_cost_usd: 0.363` for a trivial 5-turn probe (Fable 5 $0.362 + Haiku routing $0.001).
The harness reports true dollars per headless run — the morning digest can carry real cost, not
just tokens. Note for the budget conversation: even trivial ticks are ~$0.36 floor at Fable-5
pricing; a working triage tick will cost several times that. Model selection for tick sessions
(the `--model` flag / cheaper tiers for mechanical units) is a live tuning lever for Eli's
ceiling discussion.

## Stop-hook interaction

None observed — the session ended on its single short line (`probe done: psql=ok git=ok
claude-v=denied soul=ok`), `terminal_reason: completed`, no forced continuation. Prompt shaping
works.

## SCHEDULING VERDICT: **cleared.**

Re-probe path: `ops/momo-probe-tick.sh` is retained; re-run it (via a temporary guardian
request-file block) after any Claude Code CLI update — the `--bare`-default change is announced
and would silently disarm the guard if it ever applies to plain `-p`.
