# Mobile App Requirements

Status: accepted client baseline

Version: 1.0

Recorded: 12 August 2026

## Product Rationale

The mobile application was added after the original web-only scope was defined. The client identified that approximately 90% of the shop's customers are repeat customers rather than one-time visitors.

A mobile application can reduce repeat-booking friction by keeping eligible customers signed in, retaining their account details, surfacing upcoming appointments, and making subsequent bookings easier. The website remains important for search, discovery, occasional customers, and customers who do not want to install an application.

## Current Release Baseline: Mobile 1.0

Deliver a customer-only iOS and Android application that allows repeat, registered, and guest customers to discover services, create appointments, and manage bookings through the shared NestJS API.

Shared business behaviour is defined in [[Shared Product Requirements]]. This document defines how the mobile client exposes that behaviour and adds native-client requirements for session storage, application lifecycle, linking, navigation, and release delivery.

Requirements are allocated and sequenced in [[Mobile App Delivery Roadmap]]. The generic document name is intentional: this requirements catalogue will continue across Mobile 1.0 and later releases rather than being replaced for every version.

## Requirement Language

- **Shall** identifies a required mobile behaviour in the release to which the requirement is allocated.
- **Should** identifies a preferred behaviour that may require prioritisation or an open product decision.
- Shared product requirement identifiers show the business rules supported by the mobile behaviour.

## Mobile 1.0 Scope

### Included

- customer-only iOS and Android application
- public service and barber discovery
- guest and authenticated booking creation
- guest and authenticated booking management
- registration, email verification, login, MFA, password recovery, refresh, and logout
- customer profile and account deletion
- universal and Android app links with web fallback
- shop information, telephone, and map actions
- loading, empty, offline, validation, and recoverable error states

### Explicitly Deferred From Mobile 1.0

- barber, staff, and admin mobile experiences
- push notifications
- in-app payments
- service and barber administration
- barber availability management
- external-provider or social login
- loyalty points, subscriptions, and rewards
- offline booking creation
- tablet-specific layouts
- multiple-shop support

Deferred capabilities are not automatically assigned to Mobile 1.1 or Mobile 2.0. They require prioritisation, requirements, dependencies, and release allocation before they become committed scope.

## Future Staff and Administration Scope

The current mobile requirement baseline is customer-only. Mobile interfaces for barbers, other staff, or administrators are neither rejected permanently nor committed to a future version.

Before staff or administrative mobile work receives a version, the project must decide:

- whether there is sufficient business and user need for native staff or administration workflows;
- whether those users should use this application, a separately distributed application, or the responsive staff website;
- which operational tasks benefit from native mobile capabilities;
- whether the required staff and administration APIs, role enforcement, and audit controls are mature enough;
- how customer, staff, and administrative navigation and sessions would remain clearly separated;
- whether device management, stronger authentication, or other organizational controls are required; and
- what independent security, privacy, accessibility, testing, and store-distribution requirements apply.

If accepted, staff and administration work shall receive dedicated requirement identifiers, UX designs, release allocation, and any necessary ADRs. No future version number is reserved for those interfaces at this time.

## Application and Navigation Requirements

- `MOB-APP-001` — The application shall provide production-capable builds for supported iOS and Android devices.
- `MOB-APP-002` — The primary navigation shall provide access to Home, Services, Book, Bookings, and Account areas.
- `MOB-APP-003` — Customers shall not be required to log in before viewing services or starting a booking.
- `MOB-APP-004` — The application shall provide shop opening hours, location, email address, and telephone number.
- `MOB-APP-005` — Customers shall be able to initiate a telephone call using the native telephone capability.
- `MOB-APP-006` — Customers shall be able to open the shop location in an appropriate maps application.
- `MOB-APP-007` — Privacy policy and terms of service shall be accessible from the application.
- `MOB-APP-008` — Navigation state shall not be treated as evidence of authentication or authorization.
- `MOB-APP-009` — The application shall provide safe navigation after expired sessions, invalid links, and recoverable failures.

## Service and Barber Discovery Requirements

- `MOB-SVC-001` — Customers shall be able to view the active service catalogue. Supports `PROD-SVC-001` through `PROD-SVC-003`.
- `MOB-SVC-002` — Each service view shall show its name, description, applicable price, and duration. Supports `PROD-SVC-002` and `PROD-SVC-006`.
- `MOB-SVC-003` — Customers shall be able to start a booking with a service preselected.
- `MOB-SVC-004` — Customers shall be able to view active barbers available for customer booking. Supports `PROD-BAR-001` through `PROD-BAR-003`.
- `MOB-SVC-005` — Inactive services and barbers shall not be offered for a new booking.
- `MOB-SVC-006` — Service and barber data shall be obtained from the shared API rather than maintained as an independent mobile catalogue.
- `MOB-SVC-007` — A historical booking shall display its accepted service name, duration, and price snapshots rather than silently replacing them with later catalogue values. Supports `PROD-SVC-007`.

## Booking Creation Requirements

- `MOB-BOOK-001` — The application shall allow signed-in customers to create bookings associated with their account. Supports `PROD-BOOK-002` and `PROD-BOOK-005`.
- `MOB-BOOK-002` — The application shall allow customers to create bookings as guests. Supports `PROD-BOOK-001` and `PROD-BOOK-004`.
- `MOB-BOOK-003` — A booking journey shall collect an active service, eligible barber, permitted date, and available appointment time. Supports `PROD-BOOK-003`.
- `MOB-BOOK-004` — Available times shall be obtained from the shared API and displayed as provisional until booking creation succeeds. Supports `PROD-AVL-001` through `PROD-AVL-008`.
- `MOB-BOOK-005` — The application shall offer dates from tomorrow through one calendar month ahead in the shop's configured timezone and shall still rely on API validation. Supports `PROD-AVL-004` through `PROD-AVL-008`.
- `MOB-BOOK-006` — Guest customers shall be able to provide the name, email address, and phone number required by `PROD-BOOK-004`.
- `MOB-BOOK-007` — Known signed-in customer details shall be used to avoid unnecessary repeated data entry.
- `MOB-BOOK-008` — When a guest enters an email belonging to an existing account, the application shall offer login without preventing continuation as a guest.
- `MOB-BOOK-009` — A booking draft shall survive the optional login journey.
- `MOB-BOOK-010` — A recoverable booking draft should survive temporary application backgrounding.
- `MOB-BOOK-011` — Before confirmation, the application shall display the service, barber, date, time, duration, price, and customer details. Supports `PROD-SVC-006` and `PROD-BOOK-007`.
- `MOB-BOOK-012` — Booking creation shall be submitted to the shared API for final availability and integrity validation. Supports `PROD-AVL-008`, `PROD-AVL-009`, and `PROD-BOOK-012`.
- `MOB-BOOK-013` — The application shall generate one idempotency key for a booking submission, reuse it for safe retries of that submission, replace it when the booking intent changes, and also prevent repeated taps while a request is in progress. Supports `PROD-BOOK-010`.
- `MOB-BOOK-014` — If a selected slot becomes unavailable, the application shall explain the conflict and allow another slot to be selected.
- `MOB-BOOK-015` — Successful creation shall show the appointment details, status, and booking reference. Supports `PROD-BOOK-006` through `PROD-BOOK-008`.
- `MOB-BOOK-016` — No payment shall be collected in the Mobile 1.0 booking journey. Supports `PROD-BOOK-011`.

The final booking-step order remains a mobile UX decision. The current design recommends service, barber, date and time, customer details, and review because service duration and price affect availability and customer expectations.

## Booking Management Requirements

- `MOB-MGMT-001` — Signed-in customers shall be able to view their upcoming bookings. Supports `PROD-MGMT-001`.
- `MOB-MGMT-002` — Signed-in customers shall be able to view past and cancelled bookings. Supports `PROD-MGMT-002`.
- `MOB-MGMT-003` — Customers shall be able to view the service, barber, time, price, status, and permitted actions for an accessible booking.
- `MOB-MGMT-004` — Guest customers shall be able to retrieve an eligible booking using its secure reference. Supports `PROD-ACCESS-002` through `PROD-ACCESS-005`.
- `MOB-MGMT-005` — Eligible future bookings shall provide a rescheduling journey. Supports `PROD-MGMT-004` through `PROD-MGMT-006`.
- `MOB-MGMT-006` — Rescheduling shall permit service, barber, and appointment-time changes and shall present and validate current price, duration, policy, eligibility, and API availability before acceptance. Supports `PROD-MGMT-005` and `PROD-MGMT-006`.
- `MOB-MGMT-007` — Eligible future bookings shall provide a cancellation journey. Supports `PROD-MGMT-007` through `PROD-MGMT-010`.
- `MOB-MGMT-008` — Cancellation shall require explicit confirmation before the request is sent.
- `MOB-MGMT-009` — The interface shall explain when same-day, past, or already-cancelled bookings cannot be changed; same-day customers shall be directed to contact the shop. Supports `PROD-MGMT-011` and `PROD-MGMT-012`.
- `MOB-MGMT-010` — Successful rescheduling or cancellation shall display the updated booking state.
- `MOB-MGMT-011` — Booking references shall not be included in ordinary analytics events, crash reports, or client logs. Supports `PROD-PRIV-007`.
- `MOB-MGMT-012` — A Book Again action should start a new booking from an earlier booking when `PROD-MGMT-014` is included in the allocated release scope.
- `MOB-MGMT-013` — A repeated booking shall still use current service, barber, price, policy, and availability data.

## Authentication and Account Requirements

- `MOB-AUTH-001` — Customers shall be able to register with their name, email address, and password. Supports `PROD-AUTH-001` through `PROD-AUTH-003`.
- `MOB-AUTH-002` — Registration shall explain and validate the current password policy without replacing backend validation.
- `MOB-AUTH-003` — Customers shall be able to complete email verification before normal login. Supports `PROD-AUTH-004`.
- `MOB-AUTH-004` — Customers shall be able to request another verification email. Supports `PROD-AUTH-005`.
- `MOB-AUTH-005` — Verified customers shall be able to log in using email and password. Supports `PROD-AUTH-006`.
- `MOB-AUTH-006` — The application shall support the email-MFA challenge when the account requires it. Supports `PROD-AUTH-007`.
- `MOB-AUTH-007` — Customers shall be able to request and complete password recovery. Supports `PROD-AUTH-008`.
- `MOB-AUTH-008` — The application shall refresh an eligible session without unnecessarily interrupting the customer.
- `MOB-AUTH-009` — Expired, revoked, invalid, or replayed refresh sessions shall return the customer to an unauthenticated state. Supports `PROD-AUTH-010`.
- `MOB-AUTH-010` — Customers shall be able to log out manually. Supports `PROD-AUTH-009`.
- `MOB-AUTH-011` — Logout shall clear local session state even if backend invalidation cannot complete immediately.
- `MOB-AUTH-012` — Customers shall be able to view and update supported profile information. Supports `PROD-AUTH-011`.
- `MOB-AUTH-013` — Login email shall remain read-only until a verified email-change flow satisfying `PROD-AUTH-012` exists.
- `MOB-AUTH-014` — Customers shall be able to request account deletion with explicit warning and confirmation. Supports `PROD-AUTH-013`, `PROD-AUTH-014`, and `PROD-PRIV-002` through `PROD-PRIV-005`.

## Native Session Security Requirements

- `MOB-SEC-001` — The mobile client shall use an explicit native login, refresh, logout, and token-revocation contract with the NestJS API.
- `MOB-SEC-002` — The mobile client shall not depend on the Next.js BFF or its browser cookies.
- `MOB-SEC-003` — Sensitive durable refresh credentials shall be stored using Expo SecureStore or an accepted equivalent backed by operating-system secure storage.
- `MOB-SEC-004` — Access tokens and active session state shall be held in memory where practical rather than ordinary persistent storage.
- `MOB-SEC-005` — Authentication credentials shall not be stored in AsyncStorage or another unencrypted general-purpose store.
- `MOB-SEC-006` — Authentication credentials, reset tokens, verification tokens, booking references, and unnecessary personal information shall not be written to logs or analytics.
- `MOB-SEC-007` — Protected operations shall rely on backend verification and authorization rather than locally decoded claims. Supports `PROD-ACCESS-009`.
- `MOB-SEC-008` — Production API communication shall use HTTPS. Supports `PROD-NFR-003`.
- `MOB-SEC-009` — Session restoration shall fail safely when secure credentials are missing, invalid, revoked, or inaccessible.

## Universal and App Link Requirements

- `MOB-LINK-001` — Email-verification links shall open the installed application when supported.
- `MOB-LINK-002` — Password-reset links shall open the installed application when supported.
- `MOB-LINK-003` — Relevant booking-management links shall open the installed application when supported.
- `MOB-LINK-004` — Links shall fall back to an appropriate web destination when the application is not installed.
- `MOB-LINK-005` — Invalid, expired, and reused links shall display a safe recovery path.
- `MOB-LINK-006` — The application shall validate the route and expected token or reference type before performing the linked operation.
- `MOB-LINK-007` — Sensitive tokens and references shall be removed from navigation state and unnecessary retained storage after processing.
- `MOB-LINK-008` — Installed-app, uninstalled-app, expired-link, and wrong-platform scenarios shall be tested before release.
- `MOB-LINK-009` — Link handling shall preserve the distinction between customer and staff destinations.

## Repeat-Customer Experience Requirements

- `MOB-RET-001` — Returning customers shall remain signed in while their refresh session remains valid.
- `MOB-RET-002` — The signed-in Home or Bookings experience shall surface the customer's next appointment when one exists.
- `MOB-RET-003` — Known account details shall not need to be re-entered for every booking.
- `MOB-RET-004` — Customers should be able to start another booking from an earlier or upcoming booking when `PROD-MGMT-014` is included in the allocated release.
- `MOB-RET-005` — Book Again should preselect the previous service and barber when both remain eligible.
- `MOB-RET-006` — A repeat customer shall review current price, duration, policy, and availability before confirming another booking.
- `MOB-RET-007` — A booking created on the website shall appear in the mobile application when the signed-in customer is authorized to access it. Supports `PROD-CONS-004`.
- `MOB-RET-008` — A booking created in the mobile application shall remain accessible through the web client when the customer is authorized to access it.

## Application State and Reliability Requirements

- `MOB-UX-001` — Every network-backed view shall define loading, success, empty, and error states.
- `MOB-UX-002` — Recoverable failures shall provide an appropriate retry action.
- `MOB-UX-003` — The application shall clearly explain when network connectivity is unavailable.
- `MOB-UX-004` — Mobile 1.0 shall not imply that a booking can be created or changed offline.
- `MOB-UX-005` — Destructive actions shall require explicit confirmation.
- `MOB-UX-006` — Booking dates and times shall be displayed consistently in the configured shop timezone. Supports `PROD-NFR-008`.
- `MOB-UX-007` — Application foregrounding shall refresh time-sensitive information when cached data may be stale.
- `MOB-UX-008` — Session expiration during a recoverable journey should preserve non-sensitive transient progress where safe.
- `MOB-UX-009` — API validation and authorization errors shall be presented consistently while retaining stable machine-readable handling. Supports `PROD-CONS-006` and `PROD-NFR-007`.

## Mobile Accessibility Requirements

- `MOB-A11Y-001` — Core journeys shall support VoiceOver and TalkBack semantics.
- `MOB-A11Y-002` — Text shall support platform text scaling without hiding required information or actions.
- `MOB-A11Y-003` — Interactive controls shall provide accessible names, roles, states, and suitably sized touch targets.
- `MOB-A11Y-004` — Meaning shall not depend on colour alone.
- `MOB-A11Y-005` — Forms shall identify invalid fields and provide actionable recovery messages.
- `MOB-A11Y-006` — Keyboard appearance shall not obscure focused fields or primary actions.
- `MOB-A11Y-007` — Motion shall respect the platform reduced-motion preference where applicable.

## Technical and Delivery Requirements

- `MOB-TECH-001` — The application shall use React Native, Expo, and Continuous Native Generation.
- `MOB-TECH-002` — Native configuration shall initially be expressed through Expo application configuration and config plugins.
- `MOB-TECH-003` — Generated `ios` and `android` projects shall remain outside source control unless superseded by a later accepted ADR.
- `MOB-TECH-004` — The mobile application shall communicate directly with the NestJS API.
- `MOB-TECH-005` — TanStack Query shall manage server state and cache invalidation.
- `MOB-TECH-006` — React state and form state shall manage transient user-interface and booking-flow state.
- `MOB-TECH-007` — Development, test, and production API environments shall be distinguishable without application source changes.
- `MOB-TECH-008` — Supported iOS and Android versions shall be recorded before production acceptance testing.
- `MOB-TECH-009` — Core journeys shall be verified on both iOS and Android.
- `MOB-TECH-010` — Application behaviour shall be tested across fresh launch, warm foregrounding, backgrounding, and terminated-session restoration where relevant.
- `MOB-TECH-011` — Production failures shall be diagnosable without recording credentials, secure references, or unnecessary personal information.
- `MOB-TECH-012` — The application shall expose an appropriate production version and build number for support and release tracking.

## API and Release Preconditions

- `MOB-DEP-001` — The implemented NestJS API and committed OpenAPI document shall be reconciled before generated mobile API code becomes a feature dependency.
- `MOB-DEP-002` — Automated drift detection shall protect web, mobile, and API contract consistency. Supports `PROD-NFR-006`.
- `MOB-DEP-003` — The native authentication transport shall be documented and accepted before authenticated feature delivery.
- `MOB-DEP-004` — Guest booking access and reference-management endpoints shall be confirmed in the mobile-consumed API contract.
- `MOB-DEP-005` — Universal and app-link domains, routes, entitlements, Android intent filters, and web fallbacks shall be configured before link acceptance testing.
- `MOB-DEP-006` — The booking window and same-day change policy shall be agreed before booking acceptance testing.
- `MOB-DEP-007` — Privacy disclosures and application-store account-deletion obligations shall be reviewed before release.
- `MOB-DEP-008` — The supported-device policy shall be confirmed against the selected Expo and React Native versions.

## Initial Traceability

| Mobile capability | Shared requirements | UX area |
| --- | --- | --- |
| Service discovery | `PROD-SVC-001`–`PROD-SVC-006` | Services |
| Barber discovery | `PROD-BAR-001`–`PROD-BAR-005` | Booking: barber selection |
| Availability | `PROD-AVL-001`–`PROD-AVL-010` | Booking: date and time |
| Guest and account booking | `PROD-BOOK-001`–`PROD-BOOK-012` | Booking flow and confirmation |
| Booking ownership | `PROD-ACCESS-001`–`PROD-ACCESS-006` | Bookings and guest lookup |
| Reschedule and cancellation | `PROD-MGMT-001`–`PROD-MGMT-014` | Booking list and detail |
| Registration and session | `PROD-AUTH-001`–`PROD-AUTH-014` | Account and authentication |
| Email and booking links | `PROD-COM-001`–`PROD-COM-008` | Native link routes |
| Privacy and deletion | `PROD-PRIV-001`–`PROD-PRIV-007` | Account and client telemetry |
| Cross-client behaviour | `PROD-CONS-001`–`PROD-CONS-006` | All server-backed views |

The editable screen and journey map is maintained in `erics-barbers-app/Mobile App UX.excalidraw`. Its current design baseline represents the customer Mobile 1.0 release.

## Release Allocation

| Release | Requirement allocation | Interpretation |
| --- | --- | --- |
| Mobile 0.x internal builds | Foundational portions of `MOB-SEC-*`, `MOB-TECH-*`, and `MOB-DEP-*` | Internal engineering baseline rather than a public customer release. |
| Mobile 1.0 | All requirements stated with **shall**, except requirements explicitly deferred or blocked by an unresolved accepted decision | First customer-only iOS and Android release. Detailed sequencing and release gates are in [[Mobile App Delivery Roadmap]]. |
| Mobile 1.1 candidate | `MOB-MGMT-012`, `MOB-RET-004`, `MOB-RET-005`, and compatible experience improvements | Candidate repeat-customer scope. Book Again is explicitly deferred from Mobile 1.0; final Mobile 1.1 acceptance remains subject to prioritisation and Mobile 1.0 learning. |
| Later customer releases | Push notifications, payments, and other accepted customer capabilities | Requirements and version numbers remain to be decided. |
| Future staff/admin mobile | No requirements or version allocated | Requires a separate business case, requirements baseline, UX model, security analysis, and architectural decision. |

Implementation status is tracked separately from release allocation using the statuses defined in [[Mobile App Delivery Roadmap]].

## Remaining Delivery Decisions

The client baseline is accepted. Delivery still requires:

1. confirmation of the booking-step order after UX review;
2. definition of supported iOS and Android versions;
3. acceptance of the native authentication contract; and
4. journey-level acceptance criteria and verification evidence.

These delivery decisions do not reopen the booking policies accepted in [[Shared Product Requirements]].

## Version History

| Version | Date | Change |
| --- | --- | --- |
| 1.0 | 12 August 2026 | Accepted the client baseline and incorporated the Mobile 1.0 booking-window, barber-choice, same-day-change, snapshot, status, email, guest-linking, idempotency, rescheduling, and Book Again policies. |
| 0.2 | 12 August 2026 | Renamed the catalogue for use across releases, allocated the current requirements to Mobile 1.0, and recorded that future staff and administration requirements and versions remain undecided. |
| 0.1 | 12 August 2026 | Created the first consolidated customer mobile V1 requirements baseline linked to the shared product requirements and mobile UX map. |
