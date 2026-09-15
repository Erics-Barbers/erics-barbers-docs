# Known Gaps and Roadmap

Status: planning and current gaps note.

This note records what is implemented, what is incomplete, and what should be prioritized next.

Related notes:

- [[Project Overview]]
- [[Current System Architecture]]
- [[Authentication Flows]]
- [[Database Design]]
- [[Shared Product Requirements]]
- [[Web Client Requirements]]
- [[Web App Delivery Roadmap]]
- [[Mobile App Requirements]]
- [[Mobile App Delivery Roadmap]]

## Current Implementation Snapshot

The project is partially built.

Browser authentication is the most mature end-to-end capability. Customer booking creation and management have substantial web and API foundations, but need consistent polish and stronger end-to-end verification. The React Native repository is currently an Expo scaffold: Mobile 1.0 requirements, UX, architecture, roadmap, and tickets exist, while product feature implementation remains early. Staff pages are web interface foundations backed mainly by sample data. Payments, push notifications, and administrative workflows are not complete product capabilities.

Railway is the accepted target for the NestJS API, PostgreSQL, and future backend services, with isolated production and test environments. Migration from the historical Render deployment, production restoration, backup/restore verification, observability, and the operational runbook remain active delivery work under [[ADR 0026 - Use Railway For Backend Hosting]].

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

- customer booking creation and management UI exist, but need full polish and end-to-end test coverage
- staff dashboard and related staff pages use sample or static data rather than complete live workflows
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

## Mobile

Implemented foundations:

- React Native and Expo project scaffold using Expo Router and TypeScript
- iOS and Android application identifiers and local native development-build support
- Expo Continuous Native Generation workflow with generated native directories ignored
- mobile UX views and customer journey design
- accepted customer-only Mobile 1.0 requirements, delivery roadmap, and GitHub backlog
- accepted direct-to-NestJS architecture and canonical OpenAPI ownership

Known gaps:

- native authentication transport, secure credential storage, refresh, logout, and session restoration are not implemented
- generated mobile API client and stable error handling are not established
- TanStack Query, forms, application state, and environment boundaries are not established
- public discovery, guest booking, signed-in booking management, and cross-client verification are not implemented in the app
- universal/app links and web fallback are not configured
- store, signing, accessibility, telemetry, support, release, and rollback evidence is incomplete
- staff and administration mobile interfaces have no accepted scope or target release

## Delivery Roadmap

The original roadmap used six sequential phases:

1. stabilize authentication;
2. implement role enforcement;
3. build the minimum customer booking flow;
4. build barber management;
5. build administration and operations; and
6. complete production readiness.

Development did not remain sequential. Authentication, customer booking, booking management, host routing, and staff interface foundations progressed in parallel. As a result, a phase number no longer communicated either implementation status or release readiness.

The web roadmap now uses outcome-based releases:

| Release | Outcome |
| --- | --- |
| Web 0.x | Retrospective platform and authentication foundation. |
| Web 1.0 | Customer booking MVP for guest and registered customers. |
| Web 1.1 | Customer self-service and experience improvements. |
| Web 2.0 | Live, authorized barber workspace. |
| Web 3.0 | Administrative and shop operations. |

Security, authorization, API contracts, testing, accessibility, operations, and documentation are continuous workstreams with gates in every release. They are no longer postponed to a final production-readiness phase.

The full scope, gates, current cross-release evidence, and change-control model are maintained in [[Web App Delivery Roadmap]]. Requirement allocation is maintained in [[Web Client Requirements]].

## Feature Status Table

| Feature            | Status          | Notes                                                                                          |
| ------------------ | --------------- | ---------------------------------------------------------------------------------------------- |
| Registration       | Implemented     | Needs stronger test coverage.                                                                  |
| Email verification | Implemented     | Auth state after verification needs review.                                                    |
| Login              | Implemented     | Uses Next.js API route and backend auth endpoint.                                              |
| Logout             | Implemented     | Uses BFF route, clears local cookies, redirects to the homepage, and API logout is idempotent. |
| Refresh token      | Implemented     | Backend rotation plus BFF refresh handling for protected navigation and profile.               |
| Profile            | Partial         | Backend read/update exists, frontend page is minimal.                                          |
| Booking creation   | Partial         | Guest and authenticated UI/API foundations exist; polish and end-to-end verification remain.   |
| Booking management | Partial         | Lookup, listing, rescheduling, and cancellation UI/API foundations exist; verification remains. |
| Barber management  | Partial         | Data foundations and sample-backed staff pages exist; live operational workflows are missing.  |
| Role enforcement   | Partial         | The role guard and focused tests exist; unfinished modules still require complete enforcement. |
| Services           | Partial         | Customer catalogue is database-backed; administrative management is missing.                   |
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
- `Booking Flow - Planned.md`
- `Barber Dashboard - Planned.md`
- `Testing Strategy.md`
- `Deployment Guide.md` — tracked by the Railway deployment, recovery, and cutover ticket

## Guiding Principle

Keep the documentation honest:

- current docs should describe what exists
- planned docs should describe what is intended
- decision records should explain why choices were made

That will make the project easier for future developers to join without being misled by aspirational documentation.
