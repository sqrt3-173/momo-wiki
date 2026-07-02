# Skip-bin Booking Business

> GSD-style living project context (per `worksystem/gsd-source/templates/project.md`).
> Draft — reconstructed from CLAUDE.md + the roadmapper run 2026-07-02; Eli confirms at the
> `confirm_roadmap` gate. Roadmap: [[ROADMAP]] · Requirements: [[REQUIREMENTS]].

## What This Is

A booking-and-payment website plus custom CRM backend for a skip-bin hire business.
Customers book and pay for bins online against live stock; staff run bookings, customers,
inventory, and driver job routing (drop-off/pickup with accurate time windows) from the CRM.

## Core Value

A customer can book and pay for the right bin online in minutes — and the business can
actually fulfil it (never oversold, jobs routed with accurate drop-off times).

## Business Context

- **Customer**: Skip-bin hire business (Eli's client); end users = customers hiring bins + staff/drivers.
- **Revenue model**: Bin hire bookings paid online at booking time.
- **Success metric**: Bookings completed end-to-end online without phone-call fallback.

## Requirements

### Active

See [[REQUIREMENTS]] — 14 v1 requirements across AUTH / CRM / INV / BOOK / PAY / ROUTE.

### Out of Scope

(To be confirmed with Eli — candidates: multi-depot support, dynamic pricing, driver mobile app.)

## Constraints

- **Tech stack**: React/Next.js on Azure — Eli's standard stack, built for scale.
- **Inventory integrity**: live stock is a hard constraint — overbooking is a business failure, not a bug.
- **Tests**: written as part of every build (house rule).

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Vertical-slice phases (CRM+inventory → book&pay → routing) | Each phase ships observable user value; booking needs live stock to exist first | — Pending |
| Inventory as single source of truth in our backend | Overbooking prevention must be transactional, not synced-best-effort | — Pending |

---
*Last updated: 2026-07-02 after roadmap creation (stage-2 work-engine build).*
