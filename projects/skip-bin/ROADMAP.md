# Roadmap: Skip-bin booking business

> Rendered from `momo_work` (MVP v1.0) — do not hand-edit; the DB is the source of truth.

## Overview

Booking + payment site with live bin stock, custom CRM, job routing (pickup/dropoff). React/Next.js on Azure.

## Phases

- [ ] **Phase 1: CRM Foundation & Live Inventory** — Staff can securely run the CRM, and bin stock is a single live source of truth that cannot be oversold
- [ ] **Phase 2: Book & Pay** — A customer can book and pay for the right skip bin online in minutes and receive confirmation
- [ ] **Phase 3: Routing & Job Scheduling** — Every booking becomes actionable drop-off/pickup jobs with accurate time windows for customers and drivers

## Phase Details

### Phase 1: CRM Foundation & Live Inventory
**Goal**: Staff can securely run the CRM, and bin stock is a single live source of truth that cannot be oversold
**Depends on**: Nothing (first phase)
**Requirements**: AUTH-01, CRM-02, INV-01, INV-02
**Success Criteria** (what must be TRUE):
  1. Staff log in to the CRM and stay authenticated across sessions
  2. Bin stock levels reflect live backend inventory in real time
  3. A booking decrements available stock and the system refuses bookings once a size is out of stock
  4. Customer records are stored and searchable by name, address, or contact detail
**Plans**: TBD (planned during plan-phase)

### Phase 2: Book & Pay
**Goal**: A customer can book and pay for the right skip bin online in minutes and receive confirmation
**Depends on**: Phase 1
**Requirements**: BOOK-01, BOOK-02, BOOK-03, BOOK-04, PAY-01, PAY-02
**Success Criteria** (what must be TRUE):
  1. Customer sees available bin sizes with live stock and cannot select a sold-out size
  2. Customer selects a bin size and delivery date and enters their delivery address and details
  3. Customer pays online during booking and the booking only completes on successful payment
  4. Customer receives a booking confirmation plus a receipt/invoice
**Plans**: TBD (planned during plan-phase)

### Phase 3: Routing & Job Scheduling
**Goal**: Every booking becomes actionable drop-off/pickup jobs with accurate time windows for customers and drivers
**Depends on**: Phase 2
**Requirements**: ROUTE-01, ROUTE-02, ROUTE-03, CRM-01
**Success Criteria** (what must be TRUE):
  1. Each completed booking automatically generates a pickup job and a dropoff job
  2. Customer sees an accurate drop-off time window at booking and on their confirmation
  3. A driver/job schedule view lists the day's dropoff and pickup jobs
  4. Staff can view and manage bookings from the CRM
**Plans**: TBD (planned during plan-phase)

## Progress

| Phase | Plans Complete | Stage | Status |
|-------|----------------|-------|--------|
| 1. CRM Foundation & Live Inventory | 0/0 | todo | Not started |
| 2. Book & Pay | 0/0 | todo | Not started |
| 3. Routing & Job Scheduling | 0/0 | todo | Not started |

