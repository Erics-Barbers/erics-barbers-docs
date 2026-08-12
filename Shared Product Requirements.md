# Shared Product Requirements

Status: accepted baseline

Version: 1.0

Recorded: 12 August 2026

## Purpose

This document defines business capabilities and rules that apply across Eric's Barbers customer interfaces. These requirements are client-neutral: they apply whether a customer uses the web application, mobile application, or a future supported client.

The NestJS API is the enforcement boundary for requirements involving business integrity, availability, authentication, authorization, privacy, and booking lifecycle. Clients may repeat validation to improve user experience, but client-side validation does not replace backend enforcement.

Client-specific concerns belong in the relevant client requirements and architecture documentation. Examples include browser cookies and the Next.js BFF for the web client, and secure native credential storage and application links for the mobile client.

## Requirement Language

- **Shall** identifies a required product behaviour.
- **Should** identifies a preferred product behaviour that may require further prioritisation or policy agreement.
- Requirement identifiers are stable references for design, API, implementation, and test documentation.

## Customer Identity and Accounts

- `PROD-AUTH-001` — A customer shall be able to create an account using a name, email address, and password.
- `PROD-AUTH-002` — Each active account shall have a unique normalized email address.
- `PROD-AUTH-003` — Passwords shall satisfy the defined product security policy.
- `PROD-AUTH-004` — A customer shall verify ownership of their email address before normal account login.
- `PROD-AUTH-005` — A customer shall be able to request another verification email.
- `PROD-AUTH-006` — A verified customer shall be able to authenticate using their email address and password.
- `PROD-AUTH-007` — Authentication shall require MFA when MFA is enabled for the account.
- `PROD-AUTH-008` — A customer shall be able to reset a forgotten password using a time-limited, single-purpose link.
- `PROD-AUTH-009` — A customer shall be able to terminate their current session.
- `PROD-AUTH-010` — Revoked, expired, or replayed sessions shall not provide access to protected information.
- `PROD-AUTH-011` — A customer shall be able to update supported profile information.
- `PROD-AUTH-012` — Changing a login email shall require verification of the new address.
- `PROD-AUTH-013` — A customer shall be able to request account deletion.
- `PROD-AUTH-014` — Deleted accounts shall be unable to authenticate.

These requirements do not prescribe cookies, native secure storage, or another authentication transport.

## Service Catalogue

- `PROD-SVC-001` — Customers shall be able to view services currently offered by the shop.
- `PROD-SVC-002` — A service shall have a name, description, price, duration, and active state.
- `PROD-SVC-003` — Only active services shall be available for new bookings.
- `PROD-SVC-004` — The booking duration shall be derived from the selected service.
- `PROD-SVC-005` — Deactivating a service shall not remove it from historical booking records.
- `PROD-SVC-006` — A customer shall see the applicable service price and duration before confirming a booking.

Whether bookings snapshot the service name, duration, and price remains an explicit product decision. Without snapshots, changes to a service can affect how historical bookings are presented.

## Barber Catalogue

- `PROD-BAR-001` — Customers shall be able to view barbers currently accepting bookings.
- `PROD-BAR-002` — A barber shall have a public display name and active state.
- `PROD-BAR-003` — Only active barbers shall be available for new bookings.
- `PROD-BAR-004` — Deactivating a barber shall preserve their historical bookings.
- `PROD-BAR-005` — A new booking shall be assigned to an eligible barber.

Whether customers can select any available barber remains an explicit product decision.

## Availability

- `PROD-AVL-001` — Barber availability shall be calculated from recurring working rules and date-specific exceptions.
- `PROD-AVL-002` — Existing active bookings shall remove conflicting time from availability.
- `PROD-AVL-003` — Service duration shall be considered when calculating whether a slot is available.
- `PROD-AVL-004` — Availability shall be evaluated in the shop's configured timezone.
- `PROD-AVL-005` — Past appointment times shall not be offered.
- `PROD-AVL-006` — Appointment times shall fall within the permitted booking window.
- `PROD-AVL-007` — A slot shown as available shall remain provisional until the booking is successfully created.
- `PROD-AVL-008` — Final availability shall be revalidated when a booking is created or rescheduled.
- `PROD-AVL-009` — Concurrent requests shall not create conflicting bookings for the same barber.
- `PROD-AVL-010` — Cancelling a booking shall release its occupied time.

The exact future booking window must be defined as a shared business policy rather than as an independently chosen client-side constant.

## Booking Creation

- `PROD-BOOK-001` — A customer shall be able to create a booking without an account.
- `PROD-BOOK-002` — A signed-in customer shall be able to create a booking associated with their account.
- `PROD-BOOK-003` — A booking shall include an active service, eligible barber, and available appointment time.
- `PROD-BOOK-004` — A guest booking shall include the customer's name, email address, and phone number.
- `PROD-BOOK-005` — An authenticated booking shall use the authenticated customer's identity rather than trusting a submitted customer identifier.
- `PROD-BOOK-006` — A successful booking shall receive a unique, high-entropy reference.
- `PROD-BOOK-007` — A successful booking shall expose its service, barber, start time, end time, price, and status.
- `PROD-BOOK-008` — A successfully reserved and validated appointment shall initially receive the agreed booking status.
- `PROD-BOOK-009` — Failure during booking creation shall not leave a partial booking.
- `PROD-BOOK-010` — Repeated submission of the same booking request should not unintentionally create duplicate appointments.
- `PROD-BOOK-011` — No payment shall be required as part of the version 1 booking lifecycle.
- `PROD-BOOK-012` — Business-critical booking rules shall be enforced by the backend rather than only by a client.

The current implementation creates successfully validated bookings as `CONFIRMED`. The business purpose of `PENDING` must be agreed before clients depend on that state.

## Booking Access and Ownership

- `PROD-ACCESS-001` — An authenticated customer shall only access bookings associated with their account.
- `PROD-ACCESS-002` — A guest shall be able to access a booking using its secure reference.
- `PROD-ACCESS-003` — Possession of a guest booking reference shall be treated as possession of a sensitive bearer credential.
- `PROD-ACCESS-004` — Booking references shall not be predictable.
- `PROD-ACCESS-005` — A customer shall not be able to access another customer's booking using its internal database identifier.
- `PROD-ACCESS-006` — Linking a guest booking to an account shall not invalidate the existing reference unless a later accepted security decision changes this behaviour.
- `PROD-ACCESS-007` — Barber access shall be limited to bookings assigned to that barber.
- `PROD-ACCESS-008` — Administrative access shall require the appropriate authenticated role.
- `PROD-ACCESS-009` — Client-side navigation controls shall not replace backend authorization.

## Booking Management

- `PROD-MGMT-001` — An authenticated customer shall be able to view their upcoming bookings.
- `PROD-MGMT-002` — An authenticated customer shall be able to view their past and cancelled bookings.
- `PROD-MGMT-003` — A guest shall be able to manage an eligible booking using its secure reference.
- `PROD-MGMT-004` — An eligible future booking shall be reschedulable.
- `PROD-MGMT-005` — Rescheduling shall permit only supported changes to service, barber, and appointment time.
- `PROD-MGMT-006` — Rescheduling shall perform the same availability validation as booking creation.
- `PROD-MGMT-007` — An eligible future booking shall be cancellable.
- `PROD-MGMT-008` — Cancellation shall use a dedicated business operation rather than a general booking update.
- `PROD-MGMT-009` — Cancelling a booking shall record its cancellation status and audit metadata.
- `PROD-MGMT-010` — Cancelling an already-cancelled booking shall not produce an inconsistent state.
- `PROD-MGMT-011` — Past bookings shall not be rescheduled or cancelled online.
- `PROD-MGMT-012` — Same-day changes shall follow the agreed shop policy.
- `PROD-MGMT-013` — Cancelled bookings shall remain available for historical and operational reporting.
- `PROD-MGMT-014` — A repeat customer shall be able to start a new booking using details from an earlier booking, subject to current availability, pricing, and active records.

## Customer Communications

- `PROD-COM-001` — Registration shall generate an email-verification communication.
- `PROD-COM-002` — Password recovery shall generate a secure, time-limited reset communication.
- `PROD-COM-003` — A successful guest booking shall provide the customer with the reference needed to manage it.
- `PROD-COM-004` — Booking communications shall identify the appointment time, service, barber, and management route.
- `PROD-COM-005` — Rescheduling and cancellation should generate an appropriate confirmation.
- `PROD-COM-006` — Operational email delivery shall be retryable without repeating the associated business transaction.
- `PROD-COM-007` — Failure to send a confirmation email shall not silently undo a successfully committed booking.
- `PROD-COM-008` — Links shall support the appropriate web or installed-application destination.

Push notifications are outside the version 1 scope. A future push capability should consume shared booking events rather than introduce separate mobile-only business rules.

## Privacy and Retention

- `PROD-PRIV-001` — Personal information shall only be available to authorized users and operations.
- `PROD-PRIV-002` — Account deletion shall revoke all active sessions.
- `PROD-PRIV-003` — Account deletion shall anonymize customer-identifying information according to the accepted retention policy.
- `PROD-PRIV-004` — Historical booking facts required for legitimate reporting shall remain available after anonymization.
- `PROD-PRIV-005` — Historical reporting shall not depend on retained customer names or email addresses.
- `PROD-PRIV-006` — Unverified abandoned customer accounts shall be removed after the configured retention period.
- `PROD-PRIV-007` — Sensitive tokens and booking references shall not be exposed in operational logs.

## Cross-Client Consistency

- `PROD-CONS-001` — Web and mobile customers shall receive the same services, prices, durations, and availability from the shared backend.
- `PROD-CONS-002` — Booking creation, cancellation, and rescheduling rules shall behave consistently across supported clients.
- `PROD-CONS-003` — Authentication transport may differ by client, but account and session security rules shall remain consistent.
- `PROD-CONS-004` — A booking created on one client shall be available on another client when the customer has permission to access it.
- `PROD-CONS-005` — Backend contract changes shall be reflected in the documented API contract before dependent clients are released.
- `PROD-CONS-006` — Errors shall provide stable machine-readable codes where clients need different presentations of the same failure.

## Shared Non-Functional Requirements

- `PROD-NFR-001` — Business operations shall remain transactionally consistent when requests fail partway through.
- `PROD-NFR-002` — Authentication, availability, and booking operations shall be protected by appropriate rate limits.
- `PROD-NFR-003` — Production traffic shall use encrypted transport.
- `PROD-NFR-004` — Security-sensitive actions shall produce sufficient audit information without logging credentials or sensitive references.
- `PROD-NFR-005` — Core booking and authentication behaviour shall be covered by backend integration tests.
- `PROD-NFR-006` — Contract tests shall detect drift between the API implementation and its OpenAPI description.
- `PROD-NFR-007` — The shared API shall return consistent validation and authorization errors.
- `PROD-NFR-008` — Business rules involving dates shall use the configured shop timezone consistently.
- `PROD-NFR-009` — Availability and booking operations shall remain safe under concurrent requests.
- `PROD-NFR-010` — Shared customer capabilities shall not require knowledge of whether a request originated from web or mobile unless the contract explicitly requires a client-specific transport.

## Decisions Still Required

The following policy decisions remain open and may refine individual requirements without changing the overall shared-product model:

1. Define the permitted future booking window.
2. Define the same-day cancellation and rescheduling policy.
3. Decide whether customers can select any available barber.
4. Decide whether bookings snapshot service name, duration, and price.
5. Define whether `PENDING` has a version 1 business meaning.
6. Decide which booking confirmation, rescheduling, and cancellation emails are mandatory.
7. Decide whether matching-email guest bookings are linked to accounts automatically.
8. Define how accidental duplicate booking submissions are identified.
9. Decide whether a customer can change service while rescheduling.
10. Confirm whether Book Again is required for the first mobile release or a later increment.

The ordering of steps in a booking interface, such as service-first or barber-first, is a client UX decision rather than a shared business rule, provided the resulting booking satisfies the same shared requirements.

## Traceability and Change Control

Web and mobile client requirement documents should reference these requirement identifiers rather than duplicate the underlying business rules. API endpoints, UX views, and automated tests should also reference the relevant identifiers where practical.

When a requirement changes:

1. update the requirement and the document version history;
2. assess effects on the API, web client, mobile client, data model, and tests;
3. create or supersede an ADR when the change introduces a significant architectural decision; and
4. update current-state documentation when the implementation changes.

## Version History

| Version | Date | Change |
| --- | --- | --- |
| 1.0 | 12 August 2026 | Recorded the accepted shared product requirements baseline for web and mobile clients. |
