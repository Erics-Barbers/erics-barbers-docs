# ADR 0023 - Snapshot Accepted Service Terms On Bookings

Status: Accepted

Date: 2026-08-12

## Context

Services are mutable catalogue records. Their name, duration, and price can change after a customer makes a booking.

The current booking model retains a relation to the selected service but does not retain the terms accepted at booking time. Reading historical bookings through the live service relation can therefore make an earlier appointment appear to have different terms from those the customer reviewed and accepted.

Mobile 1.0 adds booking history and cross-client access, making consistent historical presentation necessary across the API, web application, mobile application, operational emails, and staff views.

## Decision

Persist immutable snapshots of the accepted service name, duration in minutes, and price in pence on each booking.

The booking shall continue to retain its service relation for catalogue traceability. New booking responses, confirmation communications, customer history, and operational views shall use the snapshots when presenting the accepted booking terms.

Rescheduling to another service, or accepting current terms for the same service during a qualifying reschedule, shall update the snapshots atomically with the booking change. General catalogue edits shall not change existing booking snapshots.

The API implementation and migration shall define an explicit compatibility strategy for existing bookings whose snapshots were not recorded. That strategy may backfill current catalogue values where available and preserve an explicit unknown value where accurate reconstruction is impossible; it must not silently claim reconstructed data is historically exact.

## Alternatives Considered

### Always Read The Current Service Record

Pros:

- no additional booking columns
- catalogue corrections appear everywhere immediately

Cons:

- historical bookings can change after confirmation
- customer emails, web, mobile, and staff views can disagree over time
- reporting cannot reliably distinguish accepted terms from current terms

### Store A Complete Serialized Service Object

Pros:

- preserves every catalogue field
- simple to capture at creation time

Cons:

- duplicates fields that are not contractual booking facts
- weak schema validation and awkward querying
- makes deliberate evolution and reporting harder

### Use A Versioned Service Catalogue

Pros:

- preserves full catalogue history
- supports auditing changes beyond bookings

Cons:

- substantially more lifecycle and query complexity
- every catalogue edit creates and manages versions
- unnecessary for the Mobile 1.0 requirement to preserve accepted booking terms

## Decision Rationale

Explicit scalar snapshots preserve the contractual facts customers and staff need while keeping the live catalogue relationship available.

This approach is straightforward to query, validate, expose through OpenAPI, and present consistently across clients. It also avoids making the full service catalogue append-only merely to protect booking history.

## Trade-Offs

Snapshot fields duplicate selected service data and require clear rules about when they can change.

The migration cannot guarantee historically accurate terms for legacy bookings if the source catalogue has already changed. That limitation must be handled explicitly in migration and release evidence.

## Consequences

Positive consequences:

- historical booking terms remain stable after catalogue edits
- emails, API responses, web, mobile, staff views, and reporting can use the same accepted facts
- service deactivation or deletion policy no longer determines historical presentation
- rescheduling can deliberately capture newly accepted terms

Negative consequences:

- the database, Prisma model, DTOs, OpenAPI contract, and clients need additional fields
- booking creation and service-changing reschedules must write snapshots transactionally
- legacy records require a documented compatibility or backfill strategy

## Follow-Up Work

- add snapshot fields and a safe migration to the API repository
- write snapshots during booking creation and qualifying rescheduling
- update API response contracts, OpenAPI, emails, web, mobile, and staff presentation
- add tests proving catalogue changes do not rewrite historical booking terms
- record the implemented legacy-booking compatibility strategy
