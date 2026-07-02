<!-- Goals page: Eli edits this directly in Obsidian; MOMO's triage reasons from it.
     A goal stated once on Discord ("goal: ...") updates this page.
     Provenance convention: machine appends carry a comment naming atom id, item id, date. -->

# Goals — NV Health

## Direction
<!-- source: CLAUDE.md "Current work" (clinic operating system); wiki/projects/bd-crm.md -->
Build and run the operating system for Eli's weight-loss clinic — custom CRM, patient
management, and service delivery — at real scale (20,000+ program appointments), and grow
the clinic on top of it. The platform is the foundation the business scales on, not an
admin tool bolted to the side.

## Constraints
<!-- source: CLAUDE.md "Environment" + soul.md "Match the stakes" -->
- Azure, built for scale; Next.js/React house stack; tests written.
- Real patients and daily clinical operations run on this — the confidence bar is high;
  foundations get refactored properly, never patched into rot.
- Health-sector legal surface (privacy, APP obligations) applies to anything patient-facing.

## Economics (what's known)
<!-- source: CLAUDE.md (20,000+ program appointments); nothing further recorded -->
- Scale signal: 20,000+ program appointments on the books.
- Revenue model, margins, CAC/LTV: **Not yet recorded — Eli to fill or state on Discord.**

## Current priorities
<!-- source: CLAUDE.md "Current work"; momo_work project rows (nv-health modules: Twilio, Calendar) -->
- Clinic OS build: multi-agent + audit-council workflows; Twilio and calendar-booking
  modules are the seeded functional areas in the work engine.
- **Not yet recorded beyond the build itself — Eli to state growth/ops priorities.**
