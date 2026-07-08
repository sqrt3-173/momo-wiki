# Feedback: report token usage as usage, never as dollars spent (Eli, 2026-07-08)

**What happened:** I reported an autonomous run as "$4.30 of its $5 budget." Eli read it as real
money being charged to API credits he never set up, and was (rightly) alarmed: "what API is this
for?!?! i don't have claude usage credits set up?!"

**The facts:** this machine runs every Claude session on Eli's SUBSCRIPTION login (OAuth,
eli@catapultagency.com.au). No API key exists in the environment, ops scripts, or shell configs
(verified 2026-07-08). `total_cost_usd` from `claude -p` is a dollar-EQUIVALENT estimate of token
usage; nothing is billed per-token. The real constraint is subscription usage/rate limits (the
Jul 3 outage), which is what --max-budget-usd actually protects.

**Why it matters:** money language triggers a hard, immediate concern for Eli — spending without
approval is his brightest line (the guard hard-blocks it). A dollar figure presented as "budget
spent" reads as exactly that violation, even when nothing was spent.

**How to apply:** report autonomous-run consumption as "usage (est. $X API-equivalent)" or plain
token counts. Reserve unqualified dollar figures for actual charges to actual payment methods.
Related: [[../ops/work-engine-v2]], heartbeat.md §6 constants note.
