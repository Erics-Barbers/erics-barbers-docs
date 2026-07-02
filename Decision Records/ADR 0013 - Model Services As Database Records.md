# ADR 0013 - Model Services As Database Records

Status: Accepted

Date: 2026-07-02

## Context

The booking flow needs services with operational data, not just names.

Each service needs:

- a customer-facing name
- a description
- a price
- a duration
- active/inactive lifecycle support

The frontend also needs the public services page and booking flow to read the same service data from the backend.

## Decision

Model services as database records in the `Service` table.

Bookings reference services through `Booking.serviceId`.

For the MVP, the customer-facing catalog is limited to:

- haircut
- beard
- haircut + beard

## Alternatives Considered

## Keep Services Hardcoded In The UI

Pros:

- fastest short-term implementation
- no service management API needed

Cons:

- frontend and backend can drift
- prices and durations cannot be trusted by the backend
- service changes require code changes
- historical booking records cannot reliably point to the selected service

## Use A Prisma Enum

Pros:

- simple type-safe values
- good fit if services never change

Cons:

- cannot attach price, duration, or description without separate mapping
- every service change becomes a schema change
- inactive services and historical reporting become awkward

## Decision Rationale

Services are business data.

Using a table lets the backend own the canonical service catalog and lets the UI render services dynamically.

It also keeps booking records tied to the exact service selected at booking time.

## Trade-Offs

The service catalog now needs seed data and eventually admin tooling.

The MVP can seed the three core services, but production operation will eventually need a controlled way for an admin to manage names, descriptions, prices, durations, and active state.

## Consequences

Positive consequences:

- services can include prices, durations, and descriptions
- frontend service pages and booking forms use backend data
- inactive services can be hidden without deleting historical links
- reporting can join bookings to services

Negative consequences:

- local development needs service seed data
- admin service management is still required for non-developer updates
- booking validation must ensure new bookings use active services

## Follow-Up Work

- build admin service management
- decide whether historical bookings should snapshot price as well as link to `Service`
- keep development seed scripts aligned with the service schema
