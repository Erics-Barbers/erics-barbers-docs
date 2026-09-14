# ADR 0024 - Make Booking Creation Idempotent

Status: Accepted

Date: 2026-08-12

## Context

Mobile networks can time out after the API has committed a booking but before the client receives the response. Customers can also repeat a submission through double taps, retries, or application lifecycle changes.

The existing database constraint prevents two active bookings for the same barber and start time, but it does not identify whether two requests express the same customer intent. A retry can therefore surface a misleading slot conflict, and a changed slot can create a second booking even though the customer meant to retry the first request.

Client-side button disabling improves the experience but cannot provide correctness across lost responses, process restarts, or concurrent requests.

## Decision

Require a client-generated idempotency key for customer booking-creation requests.

The key shall identify one booking-submission intent and shall be sent through a documented request header. The API shall persist the key, a deterministic fingerprint of the normalized booking request and authenticated or guest scope, the resulting booking reference or response data needed for replay, and bounded lifecycle metadata.

For a key seen within its supported retention period:

- the same normalized request in the same scope returns the original successful result without creating another booking;
- a request with different normalized details or scope is rejected with a stable machine-readable conflict; and
- concurrent requests using the same key are serialized by a database uniqueness guarantee so only one booking operation can succeed.

The client shall create the key when a booking submission begins, reuse it for safe retries of that unchanged submission, and replace it when the customer changes the booking intent after a terminal response or edit.

The retention period, request normalization, replayable status codes, in-progress handling, and cleanup mechanism shall be specified in the API contract before implementation is considered complete. Keys and stored fingerprints shall not contain raw credentials or unnecessary personal information.

## Alternatives Considered

### Disable The Submit Button Only

Pros:

- simple client implementation
- prevents common double taps in one running screen

Cons:

- does not handle timeouts, retries, restarts, or concurrent clients
- cannot return the original result after a lost response
- makes correctness depend on every client implementation

### Rely Only On The Barber-Slot Unique Constraint

Pros:

- already protects the core double-booking invariant
- no additional request state

Cons:

- cannot distinguish a safe retry from a new conflicting request
- returns a conflict instead of the original successful booking
- does not prevent duplicate customer intent when request details change

### Detect Duplicates Heuristically

Pros:

- no explicit client contract
- can compare email, barber, service, and nearby creation time

Cons:

- risks rejecting legitimate adjacent bookings
- produces ambiguous behaviour and difficult support cases
- cannot reliably correlate a retry with its original result

## Decision Rationale

An explicit idempotency key gives the API a stable identity for one mutation intent. Database uniqueness provides concurrency safety, while the request fingerprint prevents accidental reuse for different details.

This works consistently for web, mobile, guest, and authenticated clients without inferring client type or depending on a particular user interface.

## Trade-Offs

The API must persist and clean up idempotency records and define replay behaviour carefully.

Clients must retain the key for the lifetime of an unchanged submission. Contract and integration tests become necessary to ensure normalization and scope rules remain compatible.

## Consequences

Positive consequences:

- a lost successful response can be recovered safely
- double taps and automated retries do not create duplicate bookings
- identical retries return a useful original result instead of a misleading slot conflict
- key misuse with different booking details fails predictably
- the contract applies consistently across web and mobile clients

Negative consequences:

- booking creation requires additional persistence, header handling, and cleanup
- in-progress and replay behaviour add API complexity
- retention and privacy rules must be operated and monitored

## Follow-Up Work

- define the header, key format, retention period, normalization, scope, replay, in-progress, and cleanup rules in OpenAPI
- add database-backed idempotency persistence with a uniqueness constraint
- integrate idempotency and booking creation within a concurrency-safe transaction boundary
- return stable machine-readable errors for changed-payload reuse and unsupported keys
- update web and mobile submission flows to generate and retain keys correctly
- add integration tests for sequential retries, concurrent retries, changed payloads, lost responses, and expired keys
