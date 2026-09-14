# Mobile App Delivery Backlog

Status: accepted Mobile 1.0 execution baseline

Version: 1.2

Recorded: 12 August 2026

## Purpose

This catalogue translates [[Mobile App Delivery Roadmap]] into independently trackable GitHub issues. It gives Mobile 1.0 a complete path from contract stabilization through store-ready release while preserving traceability to [[Mobile App Requirements]] and [[Shared Product Requirements]].

The document name is intentionally release-neutral. Future accepted releases should add or revise allocations here rather than create disconnected ticket catalogues. Staff and administrative mobile interfaces remain undecided and are not included.

The private organization Project and operating rules are defined in [[Delivery Workflow]]. GitHub records live execution state; this file records the accepted backlog shape and intended sequence.

## Backlog Conventions

- Ticket IDs such as `A1` are stable catalogue identifiers, not GitHub issue numbers.
- Titles include the delivery increment and use lower-case wording where applicable.
- `P0` removes foundational uncertainty or protects release integrity; `P1` is required feature delivery; `P2` is required release completion that can normally follow the core journey.
- Size is relative: `S`, `M`, or `L`. It is not an estimate of elapsed time.
- Dependencies name catalogue IDs so they remain valid before GitHub issue numbers exist.
- One umbrella tracking issue should link every ticket below and summarize increment-level progress.

## Umbrella Tracking Issue

| ID | Repository | Title | Priority | Size | Dependencies |
| --- | --- | --- | --- | --- | --- |
| V1 | `erics-barbers-docs` | `deliver the Mobile 1.0 customer application` | P0 | L | A1–F7 |

The tracking issue should contain checklists grouped by increment and links to the requirements, roadmap, workflow, and every published issue. It reports delivery state but is not closed until all release gates pass or any deferral is explicitly accepted and documented.

## Increment A — Contracts and Application Architecture

| ID | Repository | Title | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| A1 | `erics-barbers-docs` | `[A] resolve Mobile 1.0 product policy blockers` | P0 | M | None | Booking window, same-day changes, barber selection, snapshots, status, communications, guest linking, idempotency, rescheduling, and Book Again each have an accepted answer or explicit deferral; requirements are updated. |
| A2 | `erics-barber-api` | `[A] reconcile the implemented API and OpenAPI contract` | P0 | L | A1 where policy affects contracts | Mobile-critical controllers, DTOs, responses, auth, errors, booking snapshots, and idempotency headers/results match a reproducibly generated OpenAPI document. |
| A3 | `erics-barber-api` | `[A] add automated API contract drift detection` | P0 | M | A2 | Local and CI checks fail on uncommitted contract drift and the update workflow is documented. |
| A4 | `erics-barber-api` | `[A] define and implement the native authentication contract` | P0 | L | A2; accepted architecture | Explicit native login, MFA, refresh, logout, revocation, replay, and expiry contracts work without client inference; browser BFF behaviour remains intact; ADR and tests exist. |
| A5 | `erics-barber-api` | `[A] standardize machine-readable API errors for mobile` | P0 | M | A2; coordinate A4 and A6 | Validation, auth, conflict, authorization, missing-resource, and rate-limit failures use stable documented codes and an OpenAPI-described envelope. |
| A6 | `erics-barber-api` | `[A] verify and complete guest booking management contracts` | P0 | L | A1, A2, A5 | Guest create, secure-reference lookup, reschedule, and cancellation satisfy ownership, policy, availability, timezone, and logging rules with integration tests. |
| A7 | `erics-barbers-app` | `[A] establish mobile application architecture and environments` | P0 | L | Stable public contracts; A4 interface for auth | Expo Router, TanStack Query, session, API, form, booking-draft, configuration, error, and lifecycle boundaries are documented and demonstrated by a small slice. |
| A8 | `erics-barbers-app` | `[A] make CNG development builds and quality checks reproducible` | P0 | M | None | A clean checkout can generate and run iOS and Android development builds; lint, format, type-check, and test commands pass; generated native directories remain untracked. |

## Increment B — Public Customer Application

| ID | Repository | Title | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| B1 | `erics-barbers-app` | `[B] build the customer navigation shell and visual foundation` | P1 | L | A7, A8 | Home, Services, Book, Bookings, and Account routes use a consistent theme, safe areas, typography, touch targets, and authenticated/guest presentation without treating navigation as authorization. |
| B2 | `erics-barbers-app` | `[B] deliver public service and barber discovery` | P1 | L | A2, A5, A7, B1 | Active services and barbers load from the API; price, duration, and descriptions are presented; inactive records are excluded; a service can seed booking. |
| B3 | `erics-barbers-app` | `[B] deliver home, shop information, and native contact actions` | P1 | M | B1 | Opening hours, location, email, telephone, terms, and privacy are accessible; telephone and maps actions work on iOS and Android. |
| B4 | `erics-barbers-app` | `[B] establish reusable loading, empty, offline, error, and accessibility patterns` | P1 | M | A5, A7, B1 | Public API views demonstrate consistent loading, empty, retry, offline, validation, screen-reader, text-scaling, focus, and reduced-motion behaviour. |

## Increment C — Guest Booking Lifecycle

| ID | Repository | Title | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| C1 | `erics-barbers-app` | `[C] implement the booking draft and service/barber selection` | P1 | L | A7, B2 | A typed transient draft supports active service and eligible barber selection, service preselection, navigation, validation, and temporary backgrounding. |
| C2 | `erics-barbers-app` | `[C] implement date, availability, and provisional slot selection` | P1 | L | A1, A2, A5, C1 | Permitted dates and API-provided slots respect duration, timezone, and policy; selection is clearly provisional and refreshes when stale. |
| C3 | `erics-barbers-app` | `[C] implement guest details, review, and booking submission` | P1 | L | A6, C1, C2 | Guest name, email, and phone are validated; review shows complete current details; one idempotency key is reused across safe retries; submission prevents double taps and maps validation/conflict errors safely. |
| C4 | `erics-barbers-app` | `[C] implement booking confirmation and secure reference handling` | P1 | M | C3 | Success shows status and appointment details; the reference is presented and retained only where necessary and is excluded from ordinary logs and diagnostics. |
| C5 | `erics-barbers-app` | `[C] implement guest booking lookup and details` | P1 | M | A6, B4 | A guest can enter or follow a secure reference to an eligible booking; details and permitted actions display; invalid/inaccessible references fail safely. |
| C6 | `erics-barbers-app` | `[C] implement guest rescheduling and cancellation` | P1 | L | A1, A6, C2, C5 | Eligible guest bookings can be rescheduled against current availability or cancelled with confirmation; ineligible states are explained and updated state is shown. |
| C7 | `erics-barber-api` | `[C] verify booking integrity, concurrency, and guest lifecycle end to end` | P0 | L | A1, A6 | Integration tests prove final availability revalidation, immutable accepted service terms, no conflicting concurrent bookings, transaction safety, identical-retry replay, changed-payload idempotency conflicts, reference authorization, flexible rescheduling, and cancellation. |

## Increment D — Native Accounts and Session Lifecycle

| ID | Repository | Title | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| D1 | `erics-barbers-app` | `[D] implement secure native session infrastructure` | P0 | L | A4, A5, A7 | Refresh credentials use SecureStore, access tokens remain in memory, API authorization/refresh is coordinated, restoration and expiry fail safely, and secrets never enter logs. |
| D2 | `erics-barbers-app` | `[D] implement registration, verification, and resend flows` | P1 | L | A4, D1 | A customer can register, understand password validation, enter verification-pending state, complete verification, and request another verification email. |
| D3 | `erics-barbers-app` | `[D] implement login, email MFA, restoration, and logout` | P1 | L | A4, D1 | Login and MFA establish a session; rotation, expiry, revocation, replay, cold launch, foregrounding, and local-first logout behave safely. |
| D4 | `erics-barbers-app` | `[D] implement password recovery` | P1 | M | A4, D1; F link contract may follow | A customer can request and complete recovery; invalid, expired, and reused reset material has a safe recovery path and is not retained or logged. |
| D5 | `erics-barbers-app` | `[D] implement customer profile and account deletion` | P1 | L | D1, D3 | Profile read/update works, email remains read-only, deletion requires explicit confirmation, backend anonymization/revocation is invoked, and local session data is cleared. |

## Increment E — Signed-In and Repeat-Customer Booking

| ID | Repository | Title | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| E1 | `erics-barbers-app` | `[E] implement authenticated booking and preserve drafts through login` | P1 | L | C1–C4, D1–D3 | Signed-in identity comes from the session, known details are prefilled, optional login does not lose the draft, and current details are reviewed before submission. |
| E2 | `erics-barbers-app` | `[E] surface the next appointment and authenticated booking history` | P1 | L | D1–D3; booking list contract from A2 | Home/Bookings show the next appointment plus upcoming, past, and cancelled bookings with ownership-safe detail navigation and useful empty/error states. |
| E3 | `erics-barbers-app` | `[E] implement authenticated booking rescheduling and cancellation` | P1 | L | C2, D1–D3, E2 | Eligible account bookings can be rescheduled or cancelled under the same shared policies and API validation as guest bookings. |
| E4 | `erics-barbers-app` | `[E] verify cross-client booking consistency` | P0 | M | E1–E3; Web 1.0 booking capabilities | Authorized web-created bookings appear on mobile and mobile-created/changed bookings appear on web with consistent services, prices, times, status, and actions. |

## Increment F — Application Links and Release Candidate

| ID | Repository | Title | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| F1 | `erics-barbers-ui` | `[F] host universal/app-link associations and web fallbacks` | P1 | L | Confirmed production domains and app identifiers | Valid Apple association and Android asset-links files are hosted; verification, reset, and booking routes provide safe browser fallbacks without breaking existing web flows. |
| F2 | `erics-barber-api` | `[F] emit canonical verification, reset, and booking-management links` | P1 | M | A4, A6, F1 | Operational emails use environment-aware canonical HTTPS links that can open the app or fall back to web and never expose the wrong route type. |
| F3 | `erics-barbers-app` | `[F] implement verification, reset, and booking app-link routing` | P1 | L | D2, D4, C5, F1, F2 | Cold, warm, and background link entry reaches the correct route; type, token/reference, expiry, reuse, and cleanup behaviour is safe on both platforms. |
| F4 | `erics-barbers-app` | `[F] add privacy-safe production diagnostics and support information` | P2 | M | Core journeys implemented | Production failures are diagnosable with environment/version/build context while credentials, sensitive references, and unnecessary personal data are redacted. |
| F5 | `erics-barbers-app` | `[F] complete cross-platform accessibility and lifecycle acceptance` | P0 | L | B–E journeys implemented | Core journeys pass iOS/Android checks for VoiceOver, TalkBack, text scaling, focus, touch targets, motion, loading/retry/offline, cold launch, backgrounding, and restoration on supported devices. |
| F6 | `erics-barbers-app` | `[F] configure signed store-ready builds and release versioning` | P0 | L | A8; supported-device and account decisions | EAS/build profiles, signing, identifiers, environment separation, icons/splash, permissions, privacy declarations, version/build numbers, and reproducible store artifacts are verified. |
| F7 | `erics-barbers-docs` | `[F] complete Mobile 1.0 release, rollback, support, and gate evidence` | P0 | L | A1–F6 | Every required requirement is Verified or explicitly deferred; store, monitoring, support, release, rollback, privacy, deletion, link, accessibility, and platform evidence is documented. |

## Recommended Starting Sequence

The first active ticket should be **A1** because unresolved business policy can invalidate booking acceptance work. **A2** and **A8** can begin alongside client discussion when their work does not assume an unresolved policy. After the API baseline is reconciled, prioritize **A3**, **A4**, **A5**, **A6**, and **A7**.

Public application work in Increment B can overlap later Increment A work when the public contracts are stable. Increment C proves commercial value before requiring an account. Increment D then establishes the secure repeat-customer session, Increment E joins authentication and booking, and Increment F supplies public-release evidence.

## GitHub Publication State

All 35 delivery tickets are published in their owning repositories and aggregated in the private [Erics Barbers — Mobile App Delivery Project](https://github.com/orgs/Erics-Barbers/projects/1). The [Mobile 1.0 umbrella issue](https://github.com/Erics-Barbers/erics-barbers-docs/issues/3) links every issue by increment and tracks release-level completion.

The first ticket, [A1 — resolve Mobile 1.0 product policy blockers](https://github.com/Erics-Barbers/erics-barbers-docs/issues/1), established the accepted booking baseline and unblocked the policy-dependent API and mobile work.

## Version History

| Version | Date | Change |
| --- | --- | --- |
| 1.2 | 12 August 2026 | Propagated the accepted Mobile 1.0 booking policies into snapshot, idempotency, rescheduling, and verification ticket evidence. |
| 1.1 | 12 August 2026 | Recorded publication of all 35 delivery tickets, the organization Project, the umbrella issue, and the first Ready ticket. |
| 1.0 | 12 August 2026 | Created the complete Mobile 1.0 GitHub ticket catalogue and sequencing baseline. |
