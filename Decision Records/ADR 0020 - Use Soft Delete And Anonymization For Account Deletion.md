# ADR 0020 - Use Soft Delete And Anonymization For Account Deletion

Status: Accepted

Date: 2026-07-02

## Context

Users need a way to delete their account.

The app also needs operational and accountability records. For example, barbers need to see how many bookings they handled over a period for earnings and business reporting. In many reporting cases, the customer's name is not required.

Account deletion must balance:

- user privacy
- audit and reporting needs
- referential integrity for bookings
- future right-to-erasure requests
- different consequences for customers and barbers

## Decision

Use soft delete and anonymization as the default account deletion behavior.

For customer account deletion:

- mark the `User` as deleted
- anonymize direct personal fields such as name and email
- revoke active sessions
- remove auth-adjacent state such as MFA challenges and external account links
- preserve historical booking records needed for operational reporting

For barber account deletion:

- do not silently remove booking accountability history
- deactivate the barber profile instead of erasing the operational identity immediately
- preserve booking relationships needed for reporting and business records

Hard delete is reserved for explicit erasure workflows after assessing which records can legally and operationally be removed.

## Alternatives Considered

## Hard Delete Every Account Immediately

Pros:

- simplest privacy story
- removes the user's direct account row

Cons:

- can break booking history and foreign keys
- removes operational accountability for barber work
- makes earnings and booking reports unreliable
- difficult to reconcile with records the business may need to retain

## Never Hard Delete

Pros:

- preserves all operational history
- avoids data integrity issues

Cons:

- weaker privacy posture
- does not give a path for right-to-erasure requests
- keeps more personal data than needed

## Separate Customer And Barber Deletion Policies Completely

Pros:

- highly tailored behavior per role

Cons:

- more policy surface to maintain
- easy for behavior to diverge accidentally
- still needs a common anonymization/session-revocation pattern

## Decision Rationale

Soft delete plus anonymization is the best default for this small SaaS app.

It preserves the business facts that still matter, such as:

- a booking occurred
- a barber performed work
- a service, time, and status existed

It removes or obscures personal account data that does not need to remain visible for normal reporting.

This lets the system support useful business records without keeping customer identity where it is not needed.

## Trade-Offs

Soft-deleted users must be excluded from login and normal profile lookups.

Queries need to account for `deletedAt` and anonymized records.

The app still needs a separate policy and implementation path for hard deletion when erasure is legally or operationally appropriate.

## Consequences

Positive consequences:

- users can delete their account from the UI
- deleted accounts cannot continue authenticating
- active sessions are revoked
- reports can still count historical bookings
- customer names are not required for barber earnings reporting

Negative consequences:

- deletion logic is more complex than a single database delete
- future queries must avoid exposing anonymized/deleted users incorrectly
- hard deletion remains a separate, careful workflow

## Follow-Up Work

- document the user-facing deletion policy in privacy terms
- add admin tooling for reviewing erasure requests
- define retention periods for booking and audit records
- keep tests around deleted-user login, profile access, and booking/report visibility
