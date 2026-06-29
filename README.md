# MOMO Wiki — Institutional Memory

Obsidian-style vault. Read at session start; keep current. Structured, hierarchical, linked.

## Map
- [[projects/bd-crm]] — Business Development CRM (custom Postgres, new standalone system)
- [[ops/persistence]] — always-on setup (tmux + restart loop + launchd)
- [[ops/discord-bridge]] — Discord access + the duplicate-bridge gotcha
- [[projects/clinic-os]] — Clinic operating system (Eli's weight-loss clinic) *(stub)*
- [[projects/skip-bin]] — Skip-bin booking + inventory site *(stub)*
- [[methodologies/website-audit]] — Reusable website audit checklist *(stub)*

## Environment notes
- Machine: MOMO-mini (Mac mini), user `momo`, working dir `/Users/momo/momo`.
- Toolchain (verified 2026-06-22): **bun** installed at `/Users/momo/.bun/bin/bun`.
  **No** node/npm, **no** psql/local Postgres. Build with bun (`bun install`, `bunx`).
- Reachable via Discord bridge (plugin:discord). Outbound replies require a single
  bridge instance — duplicate `claude --channels` processes break replies (see
  [[ops/discord-bridge]]).
