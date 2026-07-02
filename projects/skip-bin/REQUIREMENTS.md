# Requirements: Skip-bin Booking Business

**Defined:** 2026-07-02
**Core Value:** A customer can book and pay for the right bin online in minutes — and the business can actually fulfil it.

> Recovered verbatim from the roadmapper run (REQ-IDs live in `momo_work.phases.requirements`).
> Traceability is rendered from the DB. Project: [[PROJECT]] · Roadmap: [[ROADMAP]].

## v1 Requirements

### Authentication

- [ ] **AUTH-01**: Staff log in to the CRM.

### CRM

- [ ] **CRM-01**: Staff can view + manage bookings.
- [ ] **CRM-02**: Customer records stored + searchable.

### Inventory

- [ ] **INV-01**: Bin stock synced live from the backend inventory.
- [ ] **INV-02**: Stock decrements on booking and prevents overbooking.

### Booking

- [ ] **BOOK-01**: Customer sees available bin sizes with live stock.
- [ ] **BOOK-02**: Customer selects bin size + delivery date.
- [ ] **BOOK-03**: Customer enters delivery address + details.
- [ ] **BOOK-04**: Customer receives booking confirmation.

### Payments

- [ ] **PAY-01**: Customer pays online during booking.
- [ ] **PAY-02**: Customer receives a receipt/invoice.

### Routing

- [ ] **ROUTE-01**: Each booking generates pickup + dropoff jobs.
- [ ] **ROUTE-02**: Accurate drop-off time windows shown to the customer.
- [ ] **ROUTE-03**: Driver/job schedule view.

## v2 Requirements

(None captured yet — add as they surface.)

## Out of Scope

(To be confirmed with Eli at the `confirm_roadmap` gate.)

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| AUTH-01 | Phase 1 | Pending |
| CRM-02 | Phase 1 | Pending |
| INV-01 | Phase 1 | Pending |
| INV-02 | Phase 1 | Pending |
| BOOK-01 | Phase 2 | Pending |
| BOOK-02 | Phase 2 | Pending |
| BOOK-03 | Phase 2 | Pending |
| BOOK-04 | Phase 2 | Pending |
| PAY-01 | Phase 2 | Pending |
| PAY-02 | Phase 2 | Pending |
| ROUTE-01 | Phase 3 | Pending |
| ROUTE-02 | Phase 3 | Pending |
| ROUTE-03 | Phase 3 | Pending |
| CRM-01 | Phase 3 | Pending |

**Coverage:**
- v1 requirements: 14 total
- Mapped to phases: 14
- Unmapped: 0 ✓

---
*Requirements defined: 2026-07-02 (recovered + canonicalised during stage-2 work-engine build)*
