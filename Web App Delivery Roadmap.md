# Web App Delivery Roadmap

Status: active planning baseline

Version: 1.1

Recorded: 12 August 2026

## Purpose

This document organizes delivery of the Eric's Barbers web application into outcome-based product releases.

The project was originally planned as six sequential phases: authentication, role enforcement, customer booking, barber management, administration, and production readiness. In practice, development crossed those boundaries. Authentication continued to evolve while customer booking and staff interface foundations were also being built.

That parallel work was not inherently wrong, but the phase model stopped describing the actual state of the product. This roadmap therefore separates:

- **requirement** — the behaviour the product must provide;
- **release allocation** — the first release in which the behaviour is required;
- **implementation status** — whether evidence of that behaviour currently exists; and
- **release readiness** — whether the complete release has passed its acceptance gates.

A feature may be implemented ahead of its allocated release without moving the entire release forward. A release is complete only when all of its required scope and gates are satisfied.

Related documents:

- [[Shared Product Requirements]]
- [[Web Client Requirements]]
- [[Web App Delivery Backlog]]
- [[Known Gaps and Roadmap]]
- [[Roles and Permissions]]
- [[Authentication Flows]]
- [GitHub web delivery umbrella issue](https://github.com/Erics-Barbers/erics-barbers-docs/issues/6)

## Status Model

Requirements and release items use the following status vocabulary:

| Status | Meaning |
| --- | --- |
| Proposed | Identified but not yet accepted into a release baseline. |
| Planned | Accepted into a release but implementation has not started. |
| In progress | Some implementation evidence exists, but the requirement is not fully verified. |
| Implemented | Implementation evidence exists, but release-level verification may remain. |
| Verified | Acceptance criteria and required automated or manual checks have passed. |
| Deferred | Explicitly excluded from the current release baseline. |
| Blocked | Cannot proceed until a named dependency or decision is resolved. |

`Implemented` is not equivalent to `Verified`, and neither status by itself means that the containing release is ready.

## Release Strategy

The roadmap uses semantic product versions:

- `0.x` records the pre-release foundation already developed;
- `1.x` delivers and improves the customer web product;
- `2.0` introduces the operational barber workspace; and
- `3.0` introduces administration and shop-management capabilities.

Patch releases such as `1.0.1` are reserved for compatible fixes. A requirement moves to a later minor or major release when its product outcome is deliberately deferred, not simply because its implementation happened later than expected.

## Web 0.x — Platform and Authentication Foundation

Status: retrospective implementation baseline; not a declared production release.

### Outcome

Establish the web application, shared API integration, customer identity, browser authentication boundary, protected routing, and initial customer and staff surfaces needed by later product releases.

### Allocated Scope

- Next.js customer and staff surface foundations
- registration and email verification
- email/password login and email MFA
- password recovery
- BFF-managed HttpOnly access and refresh cookies
- refresh rotation and logout
- protected route handling and role-aware redirects
- customer profile and account deletion
- shared service catalogue foundation
- initial role and booking authorization foundations

### Primary Web Requirements

- `WEB-SURF-001`–`WEB-SURF-003`
- `WEB-AUTH-001`–`WEB-AUTH-017`
- `WEB-ACC-001`–`WEB-ACC-007`, except the verified email-change capability referenced by `WEB-ACC-003`
- `WEB-ROUTE-001`–`WEB-ROUTE-006`
- supporting `WEB-NFR-*` and `WEB-DEP-*` requirements

### Current Assessment

Most of this foundation is implemented. Remaining work is primarily verification, documentation reconciliation, full role coverage for unfinished modules, and stronger end-to-end evidence.

## Web 1.0 — Customer Booking MVP

Status: target customer release; implementation is in progress across the full release scope.

### Outcome

Allow a new, guest, or registered customer to discover the shop, create a valid appointment, and manage that appointment without staff intervention.

### Allocated Scope

- public home, service, shop-information, legal, and booking entry pages
- active service and barber discovery
- guest and authenticated booking creation
- API-calculated availability
- booking review and confirmation
- authenticated booking lists
- guest booking lookup by secure reference
- eligible rescheduling and cancellation
- responsive customer layouts
- clear loading, empty, validation, conflict, and error states
- booking integrity, ownership, and cross-client consistency from [[Shared Product Requirements]]

### Primary Web Requirements

- `WEB-SURF-004`–`WEB-SURF-009`
- `WEB-PUB-001`–`WEB-PUB-008`
- `WEB-BOOK-001`–`WEB-BOOK-013`
- `WEB-MGMT-001`–`WEB-MGMT-009`
- release-applicable `WEB-UX-*`, `WEB-NFR-*`, and `WEB-DEP-*` requirements

### Preserved MVP Policies

The accepted booking ADRs currently establish these policies unless superseded:

- the initial catalogue is haircut, beard, and haircut plus beard;
- customers select an active service and active barber;
- appointment boundaries are on the hour or half-hour;
- bookings cannot be created for today or a past date;
- bookings may be created up to one calendar month ahead;
- online cancellation and rescheduling are allowed only before the appointment's shop-local date;
- cancelled bookings do not block availability; and
- the database prevents concurrent active bookings for the same barber slot.

The variable service durations currently stored by the product and the older fixed 30-minute MVP assumption must be reconciled before Web 1.0 is verified.

### Release Gates

- shared booking policies and open decisions are resolved or explicitly deferred;
- API/OpenAPI drift affecting the release is resolved;
- guest and authenticated booking creation pass end-to-end tests;
- guest and authenticated booking management pass end-to-end tests;
- booking conflicts and stale availability are handled safely;
- customer ownership and guest-reference access are verified;
- supported-browser responsive and accessibility checks pass;
- production deployment, monitoring, recovery, and rollback checks exist for the release.

## Web 1.1 — Customer Self-Service and Experience

Status: planned after the Web 1.0 baseline.

### Outcome

Reduce friction for repeat customers and improve the quality, accessibility, and discoverability of the customer website without introducing a new operational user group.

### Candidate Scope

- Book Again, subject to acceptance of `PROD-MGMT-014`
- verified login-email change
- improved booking history and repeat-booking entry points
- consistent validation, notification, and recovery patterns
- accessibility remediation found during Web 1.0 verification
- public-page metadata, search discoverability, and performance improvements
- stronger booking communications and customer-facing status explanations

### Primary Web Requirements

- `WEB-MGMT-010`
- the complete email-change behaviour implied by `WEB-ACC-003`
- improvements arising from `WEB-PUB-007`, `WEB-UX-*`, and `WEB-NFR-008`

Items required to make Web 1.0 safe or accessible must not be deferred to Web 1.1 merely because this release contains further quality work.

## Web 2.0 — Barber Workspace

Status: planned; interface foundations exist but use sample data and are not a releasable staff product.

### Outcome

Allow an authorized barber to use live operational data to understand their working day, inspect assigned appointments, and manage their availability.

### Allocated Scope

- staff login and staff-host navigation
- role- and ownership-protected barber data
- live dashboard for today's work
- assigned booking list and details
- day or week calendar
- recurring availability rules and date-specific exceptions
- minimum necessary customer context
- supported barber preferences

### Primary Web Requirements

- `WEB-STAFF-001`–`WEB-STAFF-010`
- `PROD-ACCESS-007`–`PROD-ACCESS-009`
- applicable security, privacy, accessibility, API, and operational gates

### Release Gates

- no production staff view depends on sample data;
- barber identity is reliably linked to the authenticated account;
- cross-barber isolation is verified in API and end-to-end tests;
- availability changes affect customer slot calculation correctly;
- staff-visible personal information is limited to operational need; and
- the customer Web 1.x surface remains unaffected by staff-host changes.

## Web 3.0 — Administration and Shop Operations

Status: proposed; detailed requirements are not yet baselined.

### Outcome

Allow an authorized administrator to operate the service without routine developer intervention.

### Candidate Scope

- administrative dashboard
- service, price, duration, and active-state management
- barber onboarding, update, and deactivation
- authorized visibility and management of all bookings
- operational rescheduling and cancellation
- appropriate audit history for sensitive actions
- operational configuration supported by the product model

### Requirement Dependency

`WEB-STAFF-011` requires a dedicated administrative requirements baseline before Web 3.0 implementation can be considered complete.

## Continuous Engineering Workstreams

The following work is intentionally parallel. It is not postponed to a final production-readiness phase.

### Security and Authorization

- browser BFF and cookie security
- role and ownership enforcement
- rate limiting and abuse controls
- sensitive-data and reference handling
- security review of every release boundary

### API and Data Contracts

- OpenAPI accuracy and automated drift detection
- stable error codes and validation responses
- web/API integration tests
- data migrations and booking invariants
- cross-client consistency as the mobile client is introduced

### Quality and Accessibility

- automated unit, integration, contract, and end-to-end tests
- loading, empty, failure, and recovery states
- keyboard and assistive-technology support
- responsive layouts and supported-browser testing
- performance checks appropriate to each release

### Operations and Documentation

- environment and deployment documentation
- logging, monitoring, alerting, and health checks
- backup, recovery, release, and rollback procedures
- current-state documentation and requirement traceability

Each release defines the minimum gates it needs from these workstreams. Later work may strengthen a gate, but cannot retroactively excuse a release from basic security, integrity, accessibility, or operability.

## Current Cross-Release Implementation Evidence

Parallel development has already produced work allocated to different releases:

| Evidence | Allocated release | Current interpretation |
| --- | --- | --- |
| Customer authentication and BFF routes | Web 0.x | Implemented foundation; verification and documentation work remains. |
| Customer booking creation | Web 1.0 | Implemented foundation; polish and end-to-end coverage remain. |
| Guest lookup, rescheduling, and cancellation UI | Web 1.0 | Implemented foundation; release verification remains. |
| Staff host routing and page shell | Web 2.0 | Early implementation evidence only. |
| Staff dashboard calculations | Web 2.0 | Prototype logic backed by sample bookings, not release-ready. |
| Staff bookings, calendar, customers, availability, and settings pages | Web 2.0 | Visual foundations backed by sample or static data. |
| Payment and push capabilities | Later decision | Deferred and not part of the current web release baselines. |

This table demonstrates why implementation order must not be used as the release model.

## Requirement Allocation and Change Control

[[Web Client Requirements]] records the current requirement-to-release allocation. Detailed acceptance criteria can be added as requirements enter active release planning.

When scope changes:

1. keep the requirement identifier stable when its underlying behaviour remains the same;
2. update the allocated release separately from implementation status;
3. record the rationale when a requirement is deferred or promoted;
4. assess shared API, mobile, data, security, and test effects;
5. use an ADR when the change introduces a significant architectural decision; and
6. update current-state documentation after implementation changes.

## Version History

| Version | Date | Change |
| --- | --- | --- |
| 1.1 | 18 September 2026 | Linked the roadmap to the web execution backlog and cross-repository umbrella issue. |
| 1.0 | 12 August 2026 | Replaced sequential phases with outcome-based web releases and continuous engineering workstreams. |
