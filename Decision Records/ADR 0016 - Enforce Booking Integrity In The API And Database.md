# ADR 0016 - Enforce Booking Integrity In The API And Database

Status: Accepted

Date: 2026-07-02

## Context

Booking validation has two different responsibilities:

- provide friendly validation before writing data
- guarantee correctness when concurrent requests hit the API

The app can have concurrent requests even when only one NestJS instance is running. Two requests can overlap while awaiting database or network I/O.

Application-level availability checks are useful, but they are not enough to prevent a double booking race.

## Decision

Enforce booking integrity in both the API and the database.

The API enforces:

- active barber and service selection
- 30-minute services for the MVP
- slot start times on the hour or half-hour
- no bookings for today or past dates
- booking creation up to 1 calendar month ahead
- online reschedule and cancellation only up to the day before the appointment
- no updates to cancelled bookings
- no cancelling already-cancelled bookings

The database enforces one active booking per barber slot with a partial unique index on `Booking.barberId` and `Booking.startTime` where status is not `CANCELLED`.

## Alternatives Considered

## Rely Only On Availability Checks In The API

Pros:

- easier to implement
- friendlier error handling

Cons:

- vulnerable to concurrent double booking
- correctness depends on timing between read and write
- multiple API instances or overlapping requests can still race

## Use Pessimistic Locking For Booking Creation

Pros:

- explicit control over conflicting writes
- can work for complex booking models

Cons:

- more complex implementation
- higher chance of lock contention
- unnecessary while bookings are one barber and one 30-minute slot

## Decision Rationale

The API should provide clear validation messages, but the database must be the final source of truth for uniqueness.

The partial unique index allows cancelled bookings to remain stored for reporting while no longer blocking future availability.

## Trade-Offs

The database now contains domain-specific integrity logic.

That is acceptable because preventing double-booking is a core invariant, not just an application convenience.

## Consequences

Positive consequences:

- concurrent requests cannot double-book the same active barber slot
- cancelled bookings remain available for reporting
- API errors can map database conflicts to customer-friendly responses
- booking policy is consistently enforced on create, update, cancel, and availability lookup paths

Negative consequences:

- Prisma schema alone does not fully express the partial index
- migrations must preserve the database constraint
- future variable-duration services may require revisiting the constraint strategy

## Follow-Up Work

- add admin/staff filters for booking listings
- consider DB-level pagination for booking lists
- revisit the constraint if services later support variable durations
