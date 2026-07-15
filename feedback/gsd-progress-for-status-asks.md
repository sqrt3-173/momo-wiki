# Status asks on GSD projects → run /gsd-progress (Eli, 2026-07-08)

Eli: "When I ask 'how is everything going with X' on a project running GSD, I expect you to obviously run /gsd-progress."

**Why:** GSD's progress query reads the actual on-disk state (plans, summaries, phase dirs) — a disk-verified answer, not a from-memory summary that can drift from reality. It also routes to the next action, so "status" and "what's next" stay consistent.

**How to apply:** Any status ask about a project with a `.planning/` dir → run the gsd-progress skill first (cd into the project; call `node ~/.claude/gsd-core/bin/gsd-tools.cjs query ...` directly — var-assignment one-liners hit the guard). Blend its output with live session knowledge (e.g. in-flight subagents GSD can't see) and report both.
