# ADR 0018 - Scope Booking Access By Role And Reference

Status: Accepted

Date: 2026-07-02

## Context

The booking API supports several access patterns:

- customers view and manage their own bookings
- admins view and manage all bookings
- barbers view bookings assigned to them
- guests manage a booking by reference

It is not enough for controllers to allow a role through. The service layer still needs to apply the correct data scope for that role.

## Decision

Make booking access role-aware in the backend service/repository layer.

Authenticated booking access is scoped as follows:

- customers are scoped to `Booking.userId`
- admins can access all bookings
- barbers can access bookings assigned to their linked barber profile

Reference-based guest management is a separate bearer-reference path using the UUID booking reference.

## Alternatives Considered

## Enforce Access Only In Controllers

Pros:

- simple route-level code
- guards keep endpoint declarations readable

Cons:

- easy for repository methods to accidentally return the wrong data
- admin listing can be allowed by guards but still incorrectly scoped to the admin's own user id
- harder to test data access rules directly

## Create Separate Controllers For Customer, Admin, Barber, And Guest Booking Access

Pros:

- route ownership is explicit
- easier to shape role-specific APIs

Cons:

- duplicates booking logic
- risks inconsistent validation between roles
- heavier than the current MVP requires

## Decision Rationale

Authorization has two layers:

- route guards decide who may call an endpoint
- service/repository access scoping decides what data that caller may see or mutate

Keeping scoping in the backend service layer makes the rule testable and harder to bypass.

## Trade-Offs

The booking service methods need role and identity context, so their signatures are slightly more verbose.

That is acceptable because booking access is security-sensitive and role-dependent.

## Consequences

Positive consequences:

- customer booking listings remain limited to the authenticated customer
- admin booking listings can return all bookings
- barber access can be scoped to the linked barber profile
- guest reference management stays separate from account ownership

Negative consequences:

- role-aware methods need focused tests
- future staff/admin APIs must pass role context through consistently
- more complex filters may require a richer query object later

## Follow-Up Work

- add admin booking filters and DB-level pagination
- build staff/admin booking UI
- continue adding focused authorization tests as booking endpoints expand
