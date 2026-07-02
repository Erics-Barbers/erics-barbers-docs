# ADR 0021 - Allow Public Customer Booking Entry Points

Status: Accepted

Date: 2026-07-02

## Context

The customer booking page is a business entry point. Requiring authentication before users can view booking pages adds friction before they have chosen a service or appointment.

At the same time, booking management, account bookings, barber bookings, and admin booking workflows still need authorization.

The frontend uses a host-aware Next.js proxy to rewrite public customer routes and protect private routes.

## Decision

Allow unauthenticated users to view customer booking entry pages.

Customer-domain booking routes such as:

```text
/bookings
/bookings/new-booking
```

are treated as public customer routes and rewrite to the internal customer route folder.

Staff booking routes remain protected on the staff surface.

Account-specific booking views and backend booking operations must still enforce the correct access rules. Public page access does not imply public access to private booking data.

## Alternatives Considered

## Require Login Before Any Booking Page

Pros:

- simpler identity model
- easier to attach bookings to accounts

Cons:

- higher booking friction
- awkward for first-time customers
- conflicts with guest booking/reference-based management

## Make All Booking Routes Public

Pros:

- minimal frontend route protection
- easy for users to reach booking pages

Cons:

- unsafe for staff booking pages
- unsafe for account-specific booking history
- risks confusing public booking creation with private booking management

## Decision Rationale

Booking discovery and booking creation should be low-friction.

Authentication should be required when the user accesses account-specific data, staff tools, or privileged backend operations.

This matches the broader booking model:

- guest booking can exist without login
- authenticated customers can later see their own bookings
- booking references can act as bearer credentials for guest management
- staff/admin booking access is role-scoped in the backend

## Trade-Offs

Public booking pages need careful API boundaries.

The frontend can be public while backend endpoints still validate:

- selected service
- selected barber
- available time slot
- customer contact details
- guest reference access
- authenticated account ownership where applicable

## Consequences

Positive consequences:

- users can browse booking entry points without creating an account first
- the public site has less friction
- staff booking routes remain protected
- the route model aligns with future guest booking support

Negative consequences:

- booking UI must be clear about when login is optional versus required
- backend booking endpoints must not rely on frontend route protection
- tests must cover both customer and staff booking route behavior

## Follow-Up Work

- finish the customer booking creation flow for guest and authenticated users
- keep staff booking UI behind staff auth and role checks
- ensure account booking history uses authenticated BFF/API calls
- add end-to-end tests for public booking entry and protected booking management
