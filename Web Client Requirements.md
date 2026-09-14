# Web Client Requirements

Status: proposed client baseline

Version: 0.2

Recorded: 12 August 2026

## Purpose

This document defines requirements specific to the Eric's Barbers web client. Shared business behaviour is defined in [[Shared Product Requirements]] and is not redefined here.

The web application serves two different purposes:

1. provide a discoverable, low-friction customer website for new, occasional, guest, and registered customers; and
2. provide the browser-based foundation for future barber and administrative operations.

The customer surface is the primary implemented web product. Existing staff routes are interface foundations backed mainly by sample data, so staff requirements are identified separately as planned scope rather than described as complete functionality.

Requirement allocation follows [[Web App Delivery Roadmap]]. A release allocation states when a requirement is required for release; it does not claim that the requirement is currently implemented or verified.

## Requirement Language

- **Shall** identifies a required web-client behaviour.
- **Should** identifies a preferred behaviour that may require further prioritisation.
- Shared product requirement identifiers show the business rules supported by the web behaviour.

## Web Scope

### Included

- public customer website and shop information
- service discovery
- guest and authenticated customer booking
- customer booking management
- registration, verification, login, MFA, password recovery, session refresh, and logout
- customer profile and account deletion
- host-aware customer and staff web surfaces
- browser BFF authentication and cookie management
- responsive desktop and mobile-browser layouts
- initial staff information architecture and planned operational views

### Outside This Baseline

- native iOS and Android behaviour
- push notifications
- in-app payments
- completed administrative service and barber management workflows
- external identity-provider login while the feature remains disabled

## Surface and Navigation Requirements

- `WEB-SURF-001` — The web application shall expose a customer surface on the configured customer hostname.
- `WEB-SURF-002` — The web application shall expose a distinct staff surface on the configured staff hostname.
- `WEB-SURF-003` — Host-based routing shall not replace backend authentication or authorization.
- `WEB-SURF-004` — Public customer navigation shall provide access to Home, Services, Information, Bookings, and Account entry points.
- `WEB-SURF-005` — Customer pages shall provide consistent navigation and footer access to important product and legal information.
- `WEB-SURF-006` — Staff navigation shall not be presented as a customer capability on the customer surface.
- `WEB-SURF-007` — Customer and staff URLs shall have appropriate not-found or redirect behaviour when opened on the wrong surface.
- `WEB-SURF-008` — Public customer pages shall remain usable without an authenticated session.
- `WEB-SURF-009` — The interface shall provide visible navigation back to a safe route after recoverable errors or expired flows.

## Public Customer Website Requirements

- `WEB-PUB-001` — The website shall communicate the shop identity, location, and primary booking action.
- `WEB-PUB-002` — The website shall expose shop opening hours, address, email address, and telephone number.
- `WEB-PUB-003` — Customers shall be able to initiate a telephone call using an appropriate device link.
- `WEB-PUB-004` — Customers shall be able to open the shop location using an external map link.
- `WEB-PUB-005` — Customers shall be able to view the active service catalogue and applicable service details. Supports `PROD-SVC-001` through `PROD-SVC-006`.
- `WEB-PUB-006` — The website shall provide a privacy policy and terms of service.
- `WEB-PUB-007` — Public customer pages should be indexable and provide appropriate page metadata where business discovery benefits from search engines.
- `WEB-PUB-008` — Public content shall remain useful when authentication or protected API calls are unavailable.

## Customer Booking Requirements

- `WEB-BOOK-001` — Customers shall be able to enter the booking journey without first logging in. Supports `PROD-BOOK-001` and `PROD-ACCESS-002`.
- `WEB-BOOK-002` — The booking interface shall support both guest and authenticated booking creation. Supports `PROD-BOOK-001` through `PROD-BOOK-005`.
- `WEB-BOOK-003` — The interface shall obtain active services, eligible barbers, and availability from the shared API. Supports `PROD-SVC-003`, `PROD-BAR-003`, and `PROD-AVL-001` through `PROD-AVL-008`.
- `WEB-BOOK-004` — The interface shall display the selected service, barber, date, time, duration, price, and customer details before submission. Supports `PROD-SVC-006` and `PROD-BOOK-007`.
- `WEB-BOOK-005` — Authenticated customer details shall be used to reduce repeated data entry.
- `WEB-BOOK-006` — Guest customers shall be able to supply the details required by `PROD-BOOK-004`.
- `WEB-BOOK-007` — When an entered guest email belongs to an account, the web client shall offer login while preserving the option to continue as a guest.
- `WEB-BOOK-008` — A booking draft shall be preserved when a customer chooses to log in during the booking journey.
- `WEB-BOOK-009` — The web client shall submit the booking to the shared API for final validation rather than treating displayed availability as a reservation. Supports `PROD-AVL-007` through `PROD-AVL-009`.
- `WEB-BOOK-010` — A stale or rejected slot shall produce a recoverable response that allows the customer to select another time.
- `WEB-BOOK-011` — A successful booking shall display its confirmed details and secure booking reference. Supports `PROD-BOOK-006` through `PROD-BOOK-008`.
- `WEB-BOOK-012` — The web client shall prevent accidental repeated form submission while a booking request is in progress. Supports `PROD-BOOK-010`.
- `WEB-BOOK-013` — The customer booking interface shall not require payment in version 1. Supports `PROD-BOOK-011`.

## Customer Booking Management Requirements

- `WEB-MGMT-001` — An authenticated customer shall be able to view their upcoming, past, and cancelled bookings. Supports `PROD-MGMT-001` and `PROD-MGMT-002`.
- `WEB-MGMT-002` — A guest customer shall be able to look up a booking using its secure reference. Supports `PROD-ACCESS-002` through `PROD-ACCESS-005`.
- `WEB-MGMT-003` — Booking references shall not be placed in ordinary analytics events or client-side logs. Supports `PROD-PRIV-007`.
- `WEB-MGMT-004` — Customers shall be able to view the service, barber, time, price, status, and permitted actions for an accessible booking.
- `WEB-MGMT-005` — Eligible bookings shall provide a rescheduling journey using current API availability. Supports `PROD-MGMT-004` through `PROD-MGMT-006`.
- `WEB-MGMT-006` — Eligible bookings shall provide a cancellation journey using the dedicated cancellation operation. Supports `PROD-MGMT-007` through `PROD-MGMT-010`.
- `WEB-MGMT-007` — Cancellation shall require explicit confirmation before the request is sent.
- `WEB-MGMT-008` — The interface shall explain why an ineligible booking cannot be changed. Supports `PROD-MGMT-011` and `PROD-MGMT-012`.
- `WEB-MGMT-009` — Successful rescheduling or cancellation shall display the updated booking state.
- `WEB-MGMT-010` — The web client should provide a Book Again entry point when `PROD-MGMT-014` is included in the release scope.

## Browser Authentication and BFF Requirements

- `WEB-AUTH-001` — Browser-facing authentication requests shall pass through Next.js route handlers rather than calling NestJS authentication endpoints directly.
- `WEB-AUTH-002` — The BFF shall store access and refresh credentials in secure, HttpOnly cookies on the relevant web domain.
- `WEB-AUTH-003` — Client-side JavaScript shall not read access or refresh credentials.
- `WEB-AUTH-004` — Authentication cookies shall use production-appropriate `Secure`, `HttpOnly`, `SameSite`, path, and lifetime settings.
- `WEB-AUTH-005` — State-changing authentication routes shall reject inappropriate cross-site requests.
- `WEB-AUTH-006` — Registration shall support `PROD-AUTH-001` through `PROD-AUTH-005` without exposing authentication tokens to browser components.
- `WEB-AUTH-007` — Login shall support email and password authentication and the email-MFA step when required. Supports `PROD-AUTH-006` and `PROD-AUTH-007`.
- `WEB-AUTH-008` — Successful login and MFA verification shall set the browser session cookies through the BFF.
- `WEB-AUTH-009` — Successful email verification may establish the browser session only through the BFF cookie boundary.
- `WEB-AUTH-010` — Password-recovery pages shall support `PROD-AUTH-008` and provide safe handling for invalid or expired links.
- `WEB-AUTH-011` — The BFF shall rotate both browser credentials when the backend refresh operation succeeds.
- `WEB-AUTH-012` — Protected navigation shall attempt an eligible refresh before requiring another login.
- `WEB-AUTH-013` — Failed or invalid refresh shall clear local authentication cookies and redirect to the appropriate login route.
- `WEB-AUTH-014` — Logout shall clear browser cookies even when backend session invalidation cannot be completed.
- `WEB-AUTH-015` — Login completion shall use role-aware redirects appropriate to the customer or staff surface.
- `WEB-AUTH-016` — Frontend token decoding may inform navigation but shall not be treated as proof of authorization. Supports `PROD-ACCESS-009`.
- `WEB-AUTH-017` — External-provider login controls shall remain hidden while no complete provider flow is enabled.

## Customer Account Requirements

- `WEB-ACC-001` — A signed-in customer shall be able to view their supported profile information. Supports `PROD-AUTH-011`.
- `WEB-ACC-002` — A customer shall be able to update supported editable profile fields.
- `WEB-ACC-003` — Email shall remain read-only until a verified email-change journey satisfying `PROD-AUTH-012` exists.
- `WEB-ACC-004` — The account page shall show whether the current email address is verified.
- `WEB-ACC-005` — A customer shall be able to log out from the account area.
- `WEB-ACC-006` — Account deletion shall require an explicit warning and confirmation.
- `WEB-ACC-007` — Successful account deletion shall clear browser authentication state and leave the customer on a public route. Supports `PROD-AUTH-013`, `PROD-AUTH-014`, and `PROD-PRIV-002` through `PROD-PRIV-005`.

## Protected Routing Requirements

- `WEB-ROUTE-001` — Account-specific customer routes shall require an authenticated customer session.
- `WEB-ROUTE-002` — Public booking entry and guest-management routes shall remain reachable without login.
- `WEB-ROUTE-003` — Protected staff routes shall redirect unauthenticated users to the staff login route.
- `WEB-ROUTE-004` — An authenticated user without the required role shall not gain staff or administrative API access through route navigation.
- `WEB-ROUTE-005` — Protected API operations shall continue to enforce ownership and role rules in the NestJS API. Supports `PROD-ACCESS-001` and `PROD-ACCESS-007` through `PROD-ACCESS-009`.
- `WEB-ROUTE-006` — The route-protection configuration and route matcher shall be updated together when a new private route family is introduced.

## Planned Staff Web Requirements

The following requirements describe the intended staff web surface. The existing pages are interface foundations and shall not be represented as production-complete until they use live, authorized API data.

- `WEB-STAFF-001` — Barber and admin users shall authenticate through a staff-appropriate login entry point.
- `WEB-STAFF-002` — Staff pages shall require backend-verified staff or administrative authorization.
- `WEB-STAFF-003` — A barber shall only see bookings permitted by `PROD-ACCESS-007`.
- `WEB-STAFF-004` — The dashboard should show today's booking total, outstanding work, next appointment, and upcoming appointments using live data.
- `WEB-STAFF-005` — A barber shall be able to view assigned booking details needed to provide the service.
- `WEB-STAFF-006` — A barber should be able to view their schedule in an appropriate day or week calendar.
- `WEB-STAFF-007` — A barber should be able to manage recurring availability and date-specific exceptions once the corresponding API capability exists.
- `WEB-STAFF-008` — Customer information shown to staff shall be limited to information required for authorized operational work.
- `WEB-STAFF-009` — Staff settings shall expose only preferences supported by the backend product model.
- `WEB-STAFF-010` — Sample booking and customer data shall not be presented as production data.
- `WEB-STAFF-011` — Administrative service, barber, and all-booking workflows require a separate accepted requirements baseline before implementation is considered complete.

## Responsive Design and Accessibility Requirements

- `WEB-UX-001` — Customer journeys shall be usable on supported mobile, tablet, and desktop browser widths.
- `WEB-UX-002` — Responsive layouts shall not hide required actions or information.
- `WEB-UX-003` — Core journeys shall be operable with a keyboard.
- `WEB-UX-004` — Forms, controls, errors, statuses, and navigation shall expose appropriate accessible names and semantics.
- `WEB-UX-005` — Text and controls shall meet the agreed colour-contrast standard.
- `WEB-UX-006` — Meaning shall not depend on colour alone.
- `WEB-UX-007` — Focus shall remain visible and move appropriately during dialogs, validation, and route transitions.
- `WEB-UX-008` — Validation shall identify the affected field and explain how to recover.
- `WEB-UX-009` — Network-backed views shall define loading, empty, success, and error states.
- `WEB-UX-010` — Destructive actions shall require explicit confirmation.

## Web Reliability and Security Requirements

- `WEB-NFR-001` — Production pages and BFF routes shall be served over HTTPS.
- `WEB-NFR-002` — Browser responses shall use suitable security headers.
- `WEB-NFR-003` — Sensitive credentials and booking references shall not appear in application logs or client analytics. Supports `PROD-PRIV-007`.
- `WEB-NFR-004` — Browser authentication route handlers and protected routing shall have automated tests.
- `WEB-NFR-005` — Customer booking creation and management shall have end-to-end coverage for guest and authenticated users.
- `WEB-NFR-006` — Web-to-BFF and BFF-to-API contract behaviour shall be tested when the API changes.
- `WEB-NFR-007` — The web client shall display shared API errors consistently while preserving stable machine-readable error handling. Supports `PROD-CONS-006` and `PROD-NFR-007`.
- `WEB-NFR-008` — Public pages should provide acceptable search, loading, and interaction performance on supported browsers and ordinary mobile connections.
- `WEB-NFR-009` — Supported browser and device-width policies shall be recorded before production acceptance testing.
- `WEB-NFR-010` — Business dates and appointment times shall use the configured shop timezone. Supports `PROD-NFR-008`.

## API and Delivery Dependencies

- `WEB-DEP-001` — The web client shall depend on the NestJS API as the source of truth for shared business rules.
- `WEB-DEP-002` — The BFF shall remain the browser authentication transport boundary.
- `WEB-DEP-003` — The committed OpenAPI description shall be reconciled with the implemented API before generated client code is treated as authoritative.
- `WEB-DEP-004` — Generated API files shall not be edited manually.
- `WEB-DEP-005` — API-contract changes shall trigger regeneration and relevant web contract tests. Supports `PROD-CONS-005` and `PROD-NFR-006`.
- `WEB-DEP-006` — Environment-specific customer, staff, and API base URLs shall be configurable without source changes.

## Initial Traceability

| Web capability | Shared requirements | Primary web surface |
| --- | --- | --- |
| Service discovery | `PROD-SVC-001`–`PROD-SVC-006` | `/services` |
| Guest and account booking | `PROD-AVL-*`, `PROD-BOOK-*` | `/bookings/new-booking` |
| Booking management | `PROD-ACCESS-*`, `PROD-MGMT-*` | `/bookings/manage-booking` |
| Registration and verification | `PROD-AUTH-001`–`PROD-AUTH-005` | `/register`, `/verify-email`, `/email-verify` |
| Login, MFA, and recovery | `PROD-AUTH-006`–`PROD-AUTH-010` | `/login` and BFF routes |
| Customer account | `PROD-AUTH-011`–`PROD-AUTH-014`, `PROD-PRIV-*` | `/my-account` |
| Staff operations | `PROD-ACCESS-007`–`PROD-ACCESS-009` | Staff hostname and `/staff/*` source routes |

## Release Allocation

| Release | Requirement allocation | Release interpretation |
| --- | --- | --- |
| Web 0.x foundation | `WEB-SURF-001`–`WEB-SURF-003`, `WEB-AUTH-*`, `WEB-ACC-*`, `WEB-ROUTE-*` | Retrospective foundation. Most implementation exists, but requirements still need verification evidence. The verified email-change capability implied by `WEB-ACC-003` remains incomplete. |
| Web 1.0 customer booking MVP | `WEB-SURF-004`–`WEB-SURF-009`, `WEB-PUB-*`, `WEB-BOOK-*`, `WEB-MGMT-001`–`WEB-MGMT-009` | First complete customer booking release for guest and registered customers. |
| Web 1.1 customer self-service | `WEB-MGMT-010`, completed email-change behaviour for `WEB-ACC-003`, and accepted experience improvements | Follow-up repeat-customer and experience scope. Safety or accessibility defects required by Web 1.0 cannot be deferred here. |
| Web 2.0 barber workspace | `WEB-STAFF-001`–`WEB-STAFF-010` | Live, authorized operational barber product. Existing sample-backed pages are implementation evidence, not release completion. |
| Web 3.0 administration | Detailed baseline required by `WEB-STAFF-011` | Administrative requirements must be accepted before implementation is considered release scope. |
| Every release | Applicable `WEB-UX-*`, `WEB-NFR-*`, and `WEB-DEP-*` | Continuous release gates for accessibility, security, contracts, testing, operations, and supported environments. |

Implementation status shall be tracked separately using the statuses defined in [[Web App Delivery Roadmap]]: Proposed, Planned, In progress, Implemented, Verified, Deferred, and Blocked.

## Validation Before Acceptance

Before this proposed baseline is marked accepted:

1. confirm which staff capabilities belong to the next web release;
2. reconcile current documentation with the implemented booking-management routes;
3. decide the open shared product policies in [[Shared Product Requirements]];
4. define supported browser and accessibility targets; and
5. add acceptance criteria for each release-scoped journey.

## Version History

| Version | Date | Change |
| --- | --- | --- |
| 0.2 | 12 August 2026 | Allocated web requirement groups to outcome-based releases and separated release scope from implementation status. |
| 0.1 | 12 August 2026 | Created the first consolidated web-client requirements baseline linked to the shared product requirements. |
