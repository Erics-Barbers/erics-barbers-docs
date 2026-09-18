# Web App Delivery Backlog

Status: accepted web execution baseline

Version: 1.0

Recorded: 18 September 2026

## Purpose

This catalogue translates [[Web App Delivery Roadmap]] into independently trackable GitHub issues across the web, API, and documentation repositories. It provides a single release-oriented view of remaining work while preserving traceability to [[Web Client Requirements]] and [[Shared Product Requirements]].

Requirements and roadmaps remain the product baseline. GitHub issues record executable outcomes, the umbrella issue records cross-repository completion, and the organization Project records day-to-day status.

## Backlog Conventions

- IDs such as `W1-2` are stable catalogue identifiers, not GitHub issue numbers.
- `W0`, `W1`, `W11`, `W2`, and `W3` correspond to Web 0.x, 1.0, 1.1, 2.0, and 3.0.
- `WC` identifies continuous engineering work that can gate more than one release.
- `P0` protects release integrity or removes a blocker; `P1` is required product delivery; `P2` is required completion work that normally follows the core journey.
- Size is relative: `S`, `M`, or `L`. It is not an elapsed-time estimate.
- Dependencies use stable catalogue IDs where possible.
- Issues remain in the repository that owns the change.

## Umbrella Tracking Issue

| ID | Repository | Title | Priority | Size | Dependencies |
| --- | --- | --- | --- | --- | --- |
| WV1 | `erics-barbers-docs` | [deliver the web application roadmap](https://github.com/Erics-Barbers/erics-barbers-docs/issues/6) | P0 | L | W0-1-W3-1 and applicable continuous work |

The umbrella issue links every published ticket by release. A product release is complete only when its allocated scope and roadmap gates pass; later releases may remain open without preventing an explicitly accepted earlier release.

## Web 0.x - Platform and Authentication Foundation

| ID | Repository | Issue | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| W0-1 | `erics-barbers-ui` | [connect Vercel production and test deployments to Railway](https://github.com/Erics-Barbers/erics-barbers-ui/issues/2) | P0 | M | Backend deployment | Complete: isolated production/test routing, authentication behaviour, critical journeys, and rollback are verified. |
| W0-2 | `erics-barbers-ui` | [verify browser auth, account, and BFF foundations](https://github.com/Erics-Barbers/erics-barbers-ui/issues/3) | P0 | L | W0-1 | Registration, verification, MFA, refresh, logout, profile, deletion, protected routing, and BFF tests satisfy the allocated requirements. |

## Web 1.0 - Customer Booking MVP

| ID | Repository | Issue | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| W1-1 | `erics-barber-api` | [allow unauthenticated booking flow to read public barbers](https://github.com/Erics-Barbers/erics-barber-api/issues/11) | P0 | M | Public-field decision | Guest discovery endpoints expose only customer-safe data while private operations remain protected. |
| W1-2 | `erics-barbers-ui` | [complete customer booking creation release readiness](https://github.com/Erics-Barbers/erics-barbers-ui/issues/4) | P0 | L | W1-1; stable test data | Guest and authenticated booking creation pass against accepted policies and current API contracts, including conflict and duplicate-submit handling. |
| W1-3 | `erics-barbers-ui` | [complete customer booking management release readiness](https://github.com/Erics-Barbers/erics-barbers-ui/issues/5) | P0 | L | Stable guest-management contracts | Guest and authenticated lookup, details, rescheduling, and cancellation pass ownership, eligibility, and recovery checks. |
| W1-4 | `erics-barbers-ui` | [verify public customer website and surface routing](https://github.com/Erics-Barbers/erics-barbers-ui/issues/6) | P1 | M | W0-1 | Public pages, shop information, legal content, catalogue, navigation, host routing, metadata, and responsive states are verified. |
| W1-5 | `erics-barbers-ui` | [add customer web release acceptance coverage](https://github.com/Erics-Barbers/erics-barbers-ui/issues/7) | P0 | L | W1-2-W1-4 | Supported browsers, core journeys, accessibility, failure paths, API contract drift, and every Web 1.0 release gate have evidence or a linked defect. |
| W1-6 | `erics-barber-api` | [investigate delayed transactional email delivery](https://github.com/Erics-Barbers/erics-barber-api/issues/10) | P1 | M | Production email telemetry | Delivery latency, outbox processing, provider handoff, failure detection, and the accepted delivery objective are verified. |

## Web 1.1 - Customer Self-Service and Experience

| ID | Repository | Issue | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| W11-1 | `erics-barbers-ui` | [implement verified email change and repeat-customer improvements](https://github.com/Erics-Barbers/erics-barbers-ui/issues/8) | P1 | L | Web 1.0; accepted email-change and Book Again contracts | Verified email change and accepted repeat-customer improvements work through the BFF without weakening booking or session integrity. |

## Web 2.0 - Barber Workspace

| ID | Repository | Issue | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| W2-1 | `erics-barber-api` | [implement authorized barber workspace API contracts](https://github.com/Erics-Barbers/erics-barber-api/issues/13) | P0 | L | Web 1.0 foundations; role model | Staff identity, booking visibility, dashboard, calendar, and availability contracts are documented, authorization-tested, and represented in OpenAPI. |
| W2-2 | `erics-barbers-ui` | [replace staff sample views with live barber workspace](https://github.com/Erics-Barbers/erics-barbers-ui/issues/9) | P1 | L | W2-1 | No production staff view uses sample data; live views pass role, ownership, privacy, loading, empty, error, and customer-regression checks. |

## Web 3.0 - Administration and Shop Operations

| ID | Repository | Issue | Priority | Size | Dependencies | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| W3-1 | `erics-barbers-ui` | [define administration baseline before building admin UI](https://github.com/Erics-Barbers/erics-barbers-ui/issues/10) | P0 | L | Accepted business and authorization decisions | Admin requirements, role boundaries, destructive-action rules, privacy/audit expectations, and follow-up API/UI tickets are accepted before implementation. |

Web 3.0 implementation tickets should be created only after W3-1 establishes the baseline. This avoids presenting speculative screens as accepted product scope.

## Continuous Engineering Work

| ID | Repository | Issue | Priority | Size | Release effect | Completion evidence |
| --- | --- | --- | --- | --- | --- | --- |
| WC-1 | `erics-barber-api` and `erics-barbers-ui` | [triage and remediate dependency vulnerabilities](https://github.com/Erics-Barbers/erics-barber-api/issues/12) | P0 | L | Gates the next production release | Production-reachable critical findings are removed; high findings are fixed or accepted with mitigation; builds and tests pass. |
| WC-2 | `erics-barber-api` | [add automated API contract drift detection](https://github.com/Erics-Barbers/erics-barber-api/issues/3) | P0 | M | Shared web/mobile contract gate | CI detects uncommitted OpenAPI/client drift and the regeneration workflow is reproducible. |

## Recommended Starting Sequence

1. Complete W0-2 so browser identity and the BFF boundary have verified evidence.
2. Complete W1-1 because public discovery currently blocks the guest booking journey.
3. Run W1-2 and W1-3 against stable synthetic test data.
4. Run W1-4 in parallel when it does not depend on booking fixes.
5. Use W1-5 to close the Web 1.0 release gates and create focused defects for failures.
6. Progress W1-6 and WC-1 according to observed severity; WC-2 remains a shared contract gate.
7. Begin Web 1.1 or Web 2.0 only after Web 1.0 is accepted or an explicit parallel-work decision is recorded.

## GitHub Publication State

- All catalogue items are published in their owning repositories.
- The [web delivery umbrella issue](https://github.com/Erics-Barbers/erics-barbers-docs/issues/6) aggregates release completion.
- Completed deployment issues were closed during reconciliation so they no longer appear as outstanding work.
- The `Erics Barbers - Web App Delivery` organization Project remains to be created and populated. The available GitHub token requires `read:org`, `read:project`, and `project` scopes before Project automation can complete.

## Version History

| Version | Date | Change |
| --- | --- | --- |
| 1.0 | 18 September 2026 | Created the web backlog catalogue, mapped existing issues to releases, added a Web 2.0 API ticket, and published the umbrella issue. |
