# ADR 0014 - Model Barber Availability With Rules And Exceptions

Status: Accepted

Date: 2026-07-02

## Context

Customers choose a barber first, then choose one available slot for that barber on a specific day.

The backend needs to compute available slots from:

- the barber's normal working pattern
- one-off changes such as holidays, closures, or extra availability
- existing bookings
- the selected service duration

For the MVP, every service occupies a 30-minute slot and slots start on the hour or half-hour.

## Decision

Model barber availability with two concepts:

- weekly availability rules for normal working hours
- date-specific exceptions for one-off changes

Expose a day-by-day slot lookup endpoint for customers after they select a barber.

The API returns both a flat slot list and hourly groups so the UI can render slots by hour without duplicating grouping logic.

## Alternatives Considered

## Store Every Available Slot

Pros:

- simple reads
- direct representation of what the customer can book

Cons:

- lots of generated rows
- recurring weekly availability becomes harder to maintain
- changing a working pattern requires regenerating future slots
- one-off exceptions need careful reconciliation

## Store Only Barber Working Hours As Text

Pros:

- very simple schema
- easy to display

Cons:

- not enough structure for booking validation
- hard to subtract breaks, exceptions, and existing bookings
- pushes business rules into ad hoc parsing

## Decision Rationale

Weekly rules plus exceptions match how a barber usually works:

- normal weekly schedule
- occasional exceptions

The backend can compute availability for the requested day and remain the source of truth for booking validation.

## Trade-Offs

Availability computation is more complex than reading prebuilt slots.

The benefit is that the system can support recurring schedules, one-off exceptions, and existing booking conflicts without duplicating state.

## Consequences

Positive consequences:

- customers can view day-by-day availability
- backend validation and UI slot display use the same availability model
- exceptions can block or add availability on specific dates
- future barber availability management UI has a clear data model

Negative consequences:

- availability management endpoints and UI are still required
- slot computation needs focused tests
- future non-30-minute services would require expanding the current MVP assumption

## Follow-Up Work

- build barber availability rule management endpoints
- build exception management endpoints
- build barber/admin availability management UI
- decide how far ahead availability should be displayed for staff workflows
