# ADR 0017 - Use UUID Booking References As Bearer Credentials

Status: Accepted

Date: 2026-07-02

## Context

The product allows guests to create bookings without an account.

Those guests still need a way to view, reschedule, or cancel a booking.

The chosen management flow is:

```text
Enter booking reference -> View booking details -> Update or cancel -> Confirmation
```

If anyone with the reference can manage the booking, the reference behaves like a bearer credential.

That means references must be hard to guess.

## Decision

Use the booking UUID as the customer-facing booking reference.

Reference lookup, update, and cancellation endpoints validate references as UUID v4 values.

Reference-based management is allowed even after a guest booking is linked to a registered account. Account linking should not make the emailed reference stop working.

## Alternatives Considered

## Sequential References

Pros:

- easy for customers to read
- easy to generate

Cons:

- guessable
- enables enumeration
- weak fit for reference-as-bearer management

## Short Human-Friendly Codes

Pros:

- easier to type than UUIDs
- good customer support experience

Cons:

- must be generated with enough randomness
- needs uniqueness handling
- easy to accidentally make too short or guessable

## Require Login For All Booking Management

Pros:

- stronger customer identity binding
- account-based audit trail

Cons:

- conflicts with unauthenticated booking creation
- worse guest booking experience
- forces customers to register or log in for simple changes

## Decision Rationale

The current booking ID is already a high-entropy UUID.

Using it as the reference avoids adding another token field while still making reference guessing impractical.

Allowing reference-based management after account linking keeps the confirmation email useful and avoids surprising customers who booked as guests and later registered.

## Trade-Offs

The reference is long and less friendly to type manually.

The MVP accepts that trade-off because the reference is delivered by email and can be copied into the manage-booking flow.

## Consequences

Positive consequences:

- guest management remains possible without login
- linked bookings can still be managed from the original reference email
- references are not sequential or easily enumerable
- no extra reference column is required for the MVP

Negative consequences:

- UUIDs are not customer-friendly support codes
- anyone with the reference can manage the booking
- reference lookup endpoints need rate limiting and careful data exposure

## Follow-Up Work

- keep reference lookup endpoints rate-limited
- avoid exposing unnecessary personal data in reference lookup responses
- consider a separate friendly high-entropy reference later if support workflows need it
