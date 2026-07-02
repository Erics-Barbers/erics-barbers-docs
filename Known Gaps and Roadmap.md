# Known Gaps and Roadmap

Status: planning and current gaps note.

This note records what is implemented, what is incomplete, and what should be prioritized next.

Related notes:

- [[Project Overview]]
- [[Current System Architecture]]
- [[Authentication Flows]]
- [[Database Design]]

## Current Implementation Snapshot

The project is partially built.

The authentication flow is the main implemented feature. Booking, barber management, payments, notifications, and admin workflows are either placeholders or backend foundations.

## Implemented

## Authentication

Implemented:

- registration endpoint
- registration frontend page
- password hashing with bcrypt
- email verification email through the Resend-backed email outbox
- email verification endpoint
- email verification frontend page
- resend verification email flow
- login endpoint
- login frontend page
- access-token creation
- refresh-token creation
- session row creation in PostgreSQL
- protected backend profile endpoint
- frontend protected route proxy for account, staff booking, admin/barber-style private prefixes while customer booking entry pages remain public
- logout flow through the Next.js BFF
- refresh-token rotation
- BFF refresh handling in the proxy and profile route
- dedicated JWT token types for access, refresh, email verification, and password reset
- auth-specific rate limits
- focused tests for Next.js auth route handlers and proxy behavior
- scheduled cleanup for stale unverified customer accounts
- idempotent API logout and account-page redirect to the homepage
- transactional refresh-token session rotation
- email-code MFA challenge flow integrated into login
- scheduled cleanup for expired refresh-token sessions and MFA challenges
- refresh-token replay detection with session-family revocation
- password reset UI/BFF flow for customer and staff login views

Known gaps:

- external provider login is feature-flagged but not implemented
- role enforcement is complete only where guards have been wired; unfinished modules still need authorization work as they are built

## Database

Implemented:

- user table
- session table
- barber table
- booking table
- service table with price, duration, description, and active state
- MFA user flags and challenge table
- external account table foundation
- Prisma migrations
- generated Prisma client

Known gaps:

- bookings can reference a selected service through `serviceId`
- booking has a basic status lifecycle: `PENDING`, `CONFIRMED`, `CANCELLED`
- booking stores cancellation metadata without a customer-provided reason field
- barber availability schema and customer-facing day slot lookup endpoints exist
- barber availability management endpoints are not built yet

## Frontend

Implemented:

- home page
- navigation and footer
- register page
- login page
- login MFA code step
- verify email page
- email verification callback page
- my account page with profile read/update and logout
- services page
- booking page feature flag

Known gaps:

- initial customer booking creation UI exists, but needs full polish and test coverage
- no booking management UI
- no barber dashboard
- no admin dashboard
- no verified email-change flow from the account page
- service catalog is database-backed; admin service management is still missing
- user feedback and validation states are inconsistent across pages

## Backend

Implemented:

- NestJS app bootstrap
- Swagger setup
- CORS configuration
- helmet middleware
- cookie parser
- strict global validation pipe with DTO whitelisting and transformation
- auth module
- booking module foundation
- barbers module foundation
- health module
- payments placeholder
- notifications placeholder

Known gaps:

- booking authorization is not complete
- booking service uses generic `Error` instead of Nest exceptions
- future request DTOs must keep validation decorators complete
- payments module is placeholder-level
- notifications module is placeholder-level

## Priority Roadmap

## Phase 1: Stabilize Authentication

Goal:

Make the existing authentication flow reliable before building more features on top of it.

Tasks:

- build password reset UI/BFF flow
- keep auth documentation updated as flows change

Why this comes first:

Booking will depend on knowing who the current user is. If auth is unstable, every user-facing feature becomes harder to debug.

## Phase 2: Implement Role Enforcement

Goal:

Make customer, barber, and admin permissions real.

Tasks:

- implement `RolesGuard`
- apply role guard consistently
- include role data in access tokens if needed
- keep [[Roles and Permissions]] updated as role behavior changes
- add tests for forbidden and allowed role scenarios

Why this matters:

The app already has `@Roles(...)` decorators and a `Role` enum, but permissions are not fully enforced yet. Booking and barber workflows will need reliable authorization.

## Phase 3: Build Minimum Booking Flow

Goal:

Allow a customer to create a booking, including guest customers who are not logged in.

Tasks:

- expose database-backed services in the booking form
- build booking form on the frontend
- validate date and time input
- prevent past bookings
- prevent duplicate time slots
- require customer booking creation to select a barber
- use the barber day availability endpoint to show hourly slot groups
- prompt existing-account emails to sign in, while still allowing guest booking
- show authenticated customers their account bookings
- use the dedicated booking cancel endpoint for customer cancellations
- add backend tests for booking creation and listing

Minimum useful customer flow:

```mermaid
flowchart TD
    Barber["Select barber"] --> Time["Choose date and time"]
    Time --> Services["Select service"]
    Services --> Contact["Enter contact details"]
    Contact --> Create["Create booking"]
    Create --> Confirm["See confirmation"]
    Confirm --> List["View booking in account"]
```

## Phase 4: Barber Management

Goal:

Let the business manage barbers and let barbers see their bookings.

Tasks:

- build barber onboarding flow
- ensure creating a barber also aligns the user's role
- build barber-facing bookings view
- decide whether barbers use the same frontend or a separate subdomain
- build barber availability management endpoints and UI
- support active/inactive barber state in the UI

Design question:

Should barber features live in the same Next.js app under routes like `/barber`, or should there be a separate subdomain such as `barber.erics-barbers...`?

## Phase 5: Admin and Operations

Goal:

Give the shop owner/admin enough control to operate the application without developer intervention.

Tasks:

- admin dashboard
- manage services and prices
- manage barbers
- view all bookings
- cancel or reschedule bookings
- basic audit trail for operational actions

## Phase 6: Production Readiness

Goal:

Make the app safer and easier to run in production.

Tasks:

- deployment guide
- release checklist
- database backup plan
- logging strategy
- monitoring and alerting
- stronger error handling
- rate limiting review
- security review
- accessibility review
- end-to-end tests for core user journeys

## Feature Status Table

| Feature            | Status          | Notes                                                                                          |
| ------------------ | --------------- | ---------------------------------------------------------------------------------------------- |
| Registration       | Implemented     | Needs stronger test coverage.                                                                  |
| Email verification | Implemented     | Auth state after verification needs review.                                                    |
| Login              | Implemented     | Uses Next.js API route and backend auth endpoint.                                              |
| Logout             | Implemented     | Uses BFF route, clears local cookies, redirects to the homepage, and API logout is idempotent. |
| Refresh token      | Implemented     | Backend rotation plus BFF refresh handling for protected navigation and profile.               |
| Profile            | Partial         | Backend read/update exists, frontend page is minimal.                                          |
| Booking creation   | Partial         | Backend foundation exists, frontend placeholder.                                               |
| Booking management | Not implemented | Needs customer-facing and barber/admin-facing flows.                                           |
| Barber management  | Partial         | Backend foundation exists, frontend missing.                                                   |
| Role enforcement   | Partial         | Decorators exist, guard not implemented.                                                       |
| Services           | Partial         | Static frontend table, schema enum not connected to booking.                                   |
| Payments           | Placeholder     | Module exists but not implemented.                                                             |
| Notifications      | Placeholder     | Module exists but not implemented.                                                             |
| Health checks      | Implemented     | Database and Resend health checks exist.                                                       |

## Technical Debt

## Auth Technical Debt

- mixed frontend auth request paths should be monitored as new auth UI is added
- generated OpenAPI auth methods should stay out of browser auth flows; use generated DTO/model types where useful

## API Technical Debt

- unfinished modules still need role guards wired as they are built
- future request DTOs need explicit validation decorators for every accepted field
- generic errors should be replaced with Nest exceptions
- e2e tests are minimal
- some tests appear to reference older DTO names or use case signatures

## Frontend Technical Debt

- some route pages are placeholders
- UI styling is inconsistent in places
- auth state is intentionally centered on BFF cookies and route handlers; avoid adding direct browser-to-Nest auth calls
- admin service management UI is not built yet

## Database Technical Debt

- manual guest booking claim flow is still missing for typo or different-email cases
- barber availability needs rule and exception management endpoints
- session model may not need `barberId`

## Documentation Roadmap

Useful docs to add next:

- `Frontend Architecture.md`
- `Backend Architecture.md`
- `API Design.md`
- `Roles and Permissions.md`
- `Booking Flow - Planned.md`
- `Barber Dashboard - Planned.md`
- `Testing Strategy.md`
- `Deployment Guide.md`
- `ADR - Auth Cookie Strategy.md`
- `ADR - Booking Domain Model.md`

## Guiding Principle

Keep the documentation honest:

- current docs should describe what exists
- planned docs should describe what is intended
- decision records should explain why choices were made

That will make the project easier for future developers to join without being misled by aspirational documentation.
