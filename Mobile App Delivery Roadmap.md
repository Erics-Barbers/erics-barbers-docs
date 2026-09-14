# Mobile App Delivery Roadmap

Status: active planning baseline

Version: 1.2

Recorded: 12 August 2026

## Purpose

This document organizes delivery of the Eric's Barbers mobile application into outcome-based releases and independently verifiable increments.

[[Mobile App Requirements]] defines what the mobile product must do. This roadmap defines when requirements are needed, which dependencies must be resolved first, how work can proceed in parallel, and what evidence is required before Mobile 1.0 can be released.

The roadmap deliberately separates:

- **requirement** — the behaviour the product must provide;
- **release allocation** — the first release in which the behaviour is required;
- **implementation status** — whether implementation evidence exists; and
- **release readiness** — whether the complete release has passed its acceptance gates.

Completing an early screen does not make the containing release complete. Likewise, a later increment may begin before every earlier increment is finished when its contracts and dependencies are stable.

Related documents:

- [[Shared Product Requirements]]
- [[Mobile App Requirements]]
- [[Web App Delivery Roadmap]]
- [[Authentication Flows]]
- [[ADR 0022 - Build A Customer Mobile App With React Native And Expo]]
- [[Mobile App Delivery Backlog]]
- [[Delivery Workflow]]
- [GitHub Mobile App Delivery Project](https://github.com/orgs/Erics-Barbers/projects/1)

## Product Context

The mobile application was introduced after the original web-only scope. The client identified that approximately 90% of the shop's customers are repeat customers. Mobile 1.0 therefore focuses on lowering recurring booking friction while preserving guest access and cross-client consistency with the web application.

The customer web application remains the discovery and fallback surface. Mobile 1.0 is an additional customer client of the shared NestJS API, not a replacement for the web product.

## Status Model

| Status | Meaning |
| --- | --- |
| Proposed | Identified but not yet accepted into a release baseline. |
| Planned | Accepted into a release but implementation has not started. |
| In progress | Some implementation evidence exists, but the requirement is not fully verified. |
| Implemented | Implementation evidence exists, but release-level verification may remain. |
| Verified | Acceptance criteria and required checks have passed on supported platforms. |
| Deferred | Explicitly excluded from the current release baseline. |
| Blocked | Cannot proceed until a named dependency or decision is resolved. |

`Implemented` does not mean `Verified`, and neither status alone makes Mobile 1.0 ready for release.

## Release Strategy

| Release | Outcome |
| --- | --- |
| Mobile 0.x | Internal development builds, reproducible native generation, and architectural foundations. |
| Mobile 1.0 | First customer-only iOS and Android release for discovery, booking, accounts, and booking management. |
| Mobile 1.1 candidate | Repeat-customer convenience and compatible experience improvements informed by Mobile 1.0. |
| Later customer releases | Push notifications, payments, and other accepted customer capabilities; versions remain undecided. |
| Future staff/admin mobile | No version allocated; business need, product form, requirements, and architecture remain undecided. |

Patch releases such as `1.0.1` are reserved for compatible fixes. Deferred capabilities are not automatically assigned to the next version.

## Mobile 0.x — Internal Foundation

Status: in progress.

### Outcome

Produce reproducible iOS and Android development builds and establish the contracts and application architecture needed for feature work.

### Scope

- Expo and React Native project using Continuous Native Generation
- reproducible development builds for iOS and Android
- generated native projects kept outside source control
- development, test, and production environment model
- navigation shell and provider composition
- TanStack Query server-state foundation
- transient form and booking-draft state foundation
- secure credential-storage abstraction
- API client and error-handling conventions
- sensitive-data logging rules
- formatting, linting, type checking, and initial test harness

### Current Evidence

- the Expo project exists in `erics-barbers-app`;
- iOS and Android native projects can be generated through Expo CNG;
- the editable UX map exists at `erics-barbers-app/Mobile App UX.excalidraw`; and
- the native authentication and stable generated API contracts remain unresolved dependencies.

### Exit Evidence

- a clean checkout can create and run development builds on both platforms;
- environment configuration does not require source edits;
- generated native configuration is reproducible;
- baseline lint, type-check, and test commands pass; and
- foundational architectural decisions are documented.

## Mobile 1.0 — Customer Booking Release

Status: planned; delivery increments may proceed in parallel according to dependencies.

### Outcome

Allow a guest or registered customer to discover services, create a valid appointment, remain signed in securely when eligible, and manage accessible bookings on supported iOS and Android devices.

### Release Scope

- customer-only application navigation
- shop and service discovery
- active barber discovery
- guest and authenticated booking creation
- booking review, conflict handling, and confirmation
- guest booking management by secure reference
- authenticated booking lists and details
- eligible rescheduling and cancellation
- registration, verification, login, email MFA, password recovery, refresh, and logout
- profile management and account deletion
- universal and Android app links with web fallback
- cross-client visibility of authorized bookings
- accessibility, privacy, reliability, and release delivery requirements

The complete requirement allocation is maintained in [[Mobile App Requirements]].

## Delivery Increments

The increments below are dependency-aware delivery slices, not public versions. Work may overlap where its contracts are stable.

GitHub execution is tracked through the ticket catalogue in [[Mobile App Delivery Backlog]] and the private organization Project defined in [[Delivery Workflow]]. The roadmap remains the source of release sequence and gates; the Project records day-to-day state and handover context.

## Increment A — Contracts and Application Architecture

### Outcome

Remove the major uncertainties that would otherwise force feature rework.

### Scope

- reconcile the implemented NestJS API and OpenAPI description;
- automate contract-drift detection;
- define and accept the native login, refresh, logout, and revocation contract;
- define stable error codes required by both mobile and web clients;
- confirm guest booking reference endpoints and access rules;
- propagate the accepted booking window, same-day change, snapshot, idempotency, initial-status, email, guest-linking, and rescheduling policies into API and client contracts;
- establish API, authentication, navigation, state, forms, and secure-storage boundaries; and
- establish development, test, and production environment handling.

### Primary Requirements

- `MOB-SEC-001`–`MOB-SEC-009`
- `MOB-TECH-004`–`MOB-TECH-007`
- `MOB-DEP-001`–`MOB-DEP-004`
- relevant `PROD-CONS-*` and `PROD-NFR-*` requirements

### Exit Evidence

- native auth contract is documented and accepted;
- contract generation is repeatable and drift checks fail when expected;
- secure credential behaviour is tested independently of screens;
- API environments and error handling are documented; and
- the accepted shared policies are reflected in requirements and downstream implementation tickets.

This increment is the main dependency for authenticated feature delivery. Public view work can proceed against stable public contracts while native authentication is being finalized.

## Increment B — Public Customer Application

### Outcome

Deliver a usable public application shell that does not require an account.

### Scope

- Home and primary navigation
- service catalogue and details
- barber discovery
- shop information and opening hours
- telephone and maps actions
- privacy policy and terms
- public loading, empty, failure, retry, and offline states

### Primary Requirements

- `MOB-APP-001`–`MOB-APP-009`
- `MOB-SVC-001`–`MOB-SVC-006`
- applicable `MOB-UX-*` and `MOB-A11Y-*`

### Exit Evidence

- public data is loaded from the shared API;
- inactive services and barbers are excluded;
- public screens are usable with VoiceOver, TalkBack, and text scaling;
- telephone and maps actions behave correctly on both platforms; and
- loading, empty, retry, and offline states are demonstrated.

## Increment C — Guest Booking Lifecycle

### Outcome

Prove the complete commercial booking journey without requiring a customer account.

### Scope

- service and barber selection
- date and API availability selection
- guest contact details
- review and confirmation
- double-submit and stale-slot handling
- idempotent booking submission and retry handling
- booking reference presentation
- guest reference lookup
- eligible guest rescheduling and cancellation

### Primary Requirements

- `MOB-BOOK-002`–`MOB-BOOK-006`
- `MOB-BOOK-010`–`MOB-BOOK-016`
- `MOB-MGMT-003`–`MOB-MGMT-011`
- shared availability, booking, access, and management requirements

### Exit Evidence

- a guest can create and manage a booking on iOS and Android;
- concurrent, repeated, or stale-slot requests do not produce duplicate bookings;
- references are handled as sensitive bearer credentials;
- booking policies and timezone behaviour match the web client and API; and
- the complete guest journey has API integration and end-to-end test evidence.

## Increment D — Native Accounts and Session Lifecycle

### Outcome

Allow registered customers to securely establish, recover, restore, and end a native session.

### Scope

- registration and verification-pending state
- login and email MFA
- secure credential persistence
- access-token refresh and application session restoration
- password recovery
- profile read and update
- logout
- account deletion

### Primary Requirements

- `MOB-AUTH-001`–`MOB-AUTH-014`
- `MOB-SEC-001`–`MOB-SEC-009`
- relevant application-lifecycle requirements

### Exit Evidence

- login, MFA, refresh rotation, revocation, replay, logout, and session expiry are tested;
- sensitive durable credentials use secure operating-system storage;
- access tokens are not stored in general-purpose persistent storage;
- fresh launch, backgrounding, foregrounding, and terminated-session restoration behave safely; and
- deletion clears local state and follows the shared anonymization policy.

## Increment E — Signed-In and Repeat-Customer Booking

### Outcome

Deliver the primary repeat-customer value of the mobile application.

### Scope

- authenticated booking creation
- prefilled known customer details
- optional login during guest booking with draft restoration
- next appointment surfaced in the signed-in experience
- upcoming, past, and cancelled bookings
- authenticated booking details, rescheduling, and cancellation
- web-created bookings visible in mobile and mobile-created bookings visible on web

### Primary Requirements

- `MOB-BOOK-001`, `MOB-BOOK-007`–`MOB-BOOK-009`
- `MOB-MGMT-001`–`MOB-MGMT-003`, `MOB-MGMT-005`–`MOB-MGMT-010`, and `MOB-MGMT-013`
- `MOB-RET-001`–`MOB-RET-003`, `MOB-RET-006`–`MOB-RET-008`

### Exit Evidence

- authenticated identity is never inferred from submitted customer identifiers;
- booking progress is restored safely after optional login;
- account booking lists obey ownership rules;
- repeat customers do not need to re-enter known details unnecessarily;
- cross-client booking visibility is verified; and
- authenticated booking journeys have integration and end-to-end coverage on both platforms.

## Increment F — Application Links and Release Candidate

### Outcome

Complete native entry routes and demonstrate that the application is safe and supportable for public distribution.

### Scope

- iOS universal links
- Android app links
- email verification route
- password-reset route
- booking-management route
- installed-app and web-fallback behaviour
- invalid, expired, reused, and wrong-platform link recovery
- production telemetry and privacy review
- accessibility and supported-device verification
- signing, versioning, build, store, release, rollback, and support procedures

### Primary Requirements

- `MOB-LINK-001`–`MOB-LINK-009`
- `MOB-TECH-008`–`MOB-TECH-012`
- `MOB-DEP-005`–`MOB-DEP-008`
- all Mobile 1.0 release gates

Link routing and domain configuration should begin before this increment. This increment is where all link scenarios become release evidence.

### Exit Evidence

- installed and uninstalled link scenarios pass on iOS and Android;
- sensitive link material is not unnecessarily retained;
- supported platform versions and devices are documented and tested;
- privacy disclosures and account deletion satisfy release obligations;
- production diagnostics exclude credentials, secure references, and unnecessary personal data;
- signed store-ready artifacts can be reproduced; and
- release, rollback, and support procedures are documented.

## Mobile 1.0 Release Gates

Mobile 1.0 may be released only when:

- every required Mobile 1.0 requirement is Verified or has an explicitly accepted deferral;
- API/OpenAPI drift affecting the application is resolved and automated detection is active;
- the native authentication contract and secure storage behaviour are accepted and verified;
- guest and authenticated booking creation and management pass on iOS and Android;
- booking integrity, ownership, concurrency, timezone, and same-day policies are verified;
- accepted service terms remain stable in booking history after catalogue changes;
- repeated booking-creation requests satisfy the idempotency contract;
- universal and app links pass installed-app and web-fallback scenarios;
- accessibility checks cover the complete core journeys;
- privacy, account deletion, telemetry, and sensitive-data handling are reviewed;
- production monitoring, support, release, and rollback procedures exist; and
- no staff or administrative interface is exposed as an incomplete customer capability.

## Mobile 1.1 Candidate Scope

Mobile 1.1 is not yet an accepted baseline. Book Again is explicitly deferred from Mobile 1.0 and retained here as a candidate alongside:

- Book Again using an earlier booking;
- eligible service and barber preselection;
- improved booking-history and repeat-booking entry points;
- add-to-calendar convenience;
- compatible experience, performance, and accessibility improvements informed by Mobile 1.0 use.

Safety, integrity, privacy, or accessibility work required to make Mobile 1.0 usable cannot be deferred merely because Mobile 1.1 contains additional quality improvements.

## Later Customer Releases

The following capabilities require separate requirements, prioritisation, and architectural decisions before a version is allocated:

- push notification token registration, preferences, reminders, and operational delivery;
- payments, refunds, failure recovery, and financial reconciliation;
- loyalty, subscriptions, or rewards;
- expanded offline behaviour;
- tablet-specific experiences; and
- multiple-shop support.

Push notifications and payments are known future possibilities, not committed release numbers.

## Future Staff and Administration Mobile Interfaces

No mobile version is allocated to barber, staff, or administrative interfaces.

Before such work enters this roadmap, the project must decide:

1. whether native mobile provides sufficient benefit over the responsive staff website;
2. whether staff and administration belong in the customer application or a separately distributed application;
3. which user roles and operational journeys are in scope;
4. whether the backend authorization, availability-management, administration, and audit APIs are ready;
5. what stronger authentication, device, privacy, and organizational controls apply;
6. how customer and operational experiences remain clearly separated; and
7. how distribution, support, and application-store policies differ from the customer release.

An accepted business case must be followed by dedicated requirements, UX designs, threat analysis, release allocation, and ADRs where necessary. The roadmap intentionally does not assume that staff or administration will be Mobile 2.0.

## Continuous Engineering Workstreams

These workstreams run throughout Mobile 0.x and Mobile 1.0 rather than waiting for the final increment.

### Security and Privacy

- native credential lifecycle and revocation
- sensitive-data logging and telemetry controls
- guest-reference security
- application-link token handling
- privacy disclosures and account deletion

### API and Cross-Client Contracts

- OpenAPI accuracy and drift detection
- stable error codes and validation behaviour
- web/mobile consistency
- schema and migration effects
- backend integration tests

### Quality and Accessibility

- unit, integration, contract, and end-to-end testing
- iOS and Android behaviour
- loading, empty, offline, error, and recovery states
- VoiceOver, TalkBack, text scaling, keyboard, touch target, and motion support
- application lifecycle and supported-device testing

### Build, Release, and Operations

- reproducible Expo CNG builds
- environment, signing, version, and build-number management
- production diagnostics and support information
- store submission and release procedures
- monitoring, rollback, and incident handling
- current-state documentation and requirement traceability

## Requirement and Roadmap Change Control

When scope changes:

1. keep the requirement identifier stable when its underlying behaviour remains the same;
2. update release allocation separately from implementation status;
3. record why a requirement is promoted, deferred, or removed;
4. assess API, web, data, privacy, security, link, and test effects;
5. create or supersede an ADR for significant architectural decisions; and
6. update current-state documentation after implementation changes.

## Version History

| Version | Date | Change |
| --- | --- | --- |
| 1.2 | 12 August 2026 | Recorded the accepted Mobile 1.0 booking policies and added snapshot and idempotency evidence to delivery and release gates. |
| 1.1 | 12 August 2026 | Linked the roadmap to the assistant-managed GitHub execution workflow and Mobile 1.0 backlog catalogue. |
| 1.0 | 12 August 2026 | Created the Mobile 1.0 delivery roadmap with dependency-aware increments, release gates, continuous workstreams, and undecided future staff and administration scope. |
