---
name: wiki-capture
description: At the end of every meaningful job, capture learnings into the wiki — structured, linked, pruned. The protocol that keeps institutional memory alive.
---

# wiki-capture

The reflex that turns work into durable memory. Fires at the **end of every meaningful job** — not
as an afterthought, as a step. Without it, each session's learning dies with the session.

## When to use
- Finished a research sweep, a build, an audit, a debugging session, or made an architectural decision.
- Learned a non-obvious fact, a failure mode, or a reusable method.
- **Skip** for trivial/conversational turns — capturing noise is worse than capturing nothing.

## Inputs
- What was done, what was decided (and *why*), what was learned, what surprised you.
- The current wiki structure (so the note lands in the right place and links correctly).

## Method
1. **Decide if it's worth capturing.** Durable + reusable + non-obvious = yes. One-off + conversational = no. Curation over volume.
2. **Find the right home.** Update an existing note if one covers the topic — don't create a near-duplicate. Else create a new note under the right section (`projects/`, `ops/`, `skills/`).
3. **Write it structured:** lead with the decision/answer; capture the *why*, not just the *what*; convert relative dates to absolute; label inferences vs facts; flag confidence on uncertain claims.
4. **Link it.** Use `[[note-name]]` to wire it into related notes — liberally. An unlinked note is hard to find again.
5. **Append gotchas to the relevant skill** if the job taught a new failure mode.
6. **Commit** (and once the remote is live, push): clear message describing the learning. Tie capture → commit so memory and history stay in sync.
7. **Periodic prune** (separate pass, ~weekly or when a section bloats): merge duplicates, delete wrong/stale notes, tighten. Unnavigable memory = no memory.

## Output contract
- A new or updated wiki note, structured + linked, committed to the wiki git repo.
- Any new failure mode appended to the relevant skill's Gotchas.

## Gotchas
- **Bloat is the enemy, not the goal.** "Always building" must not become auto-generated noise. Capture-on-completion + prune, never a firehose.
- **Don't duplicate the repo.** Don't capture what code/git history/CLAUDE.md already records. If asked to "remember" such a thing, capture what was *non-obvious* about it instead.
- **Secrets never go in the wiki** (the `.gitignore` blocks common patterns, but stay deliberate).
- **Recalled notes reflect when they were written** — verify a named file/flag/function still exists before acting on an old note.
