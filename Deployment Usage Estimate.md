# Deployment Usage Estimate

Status: Initial planning estimate

Date: 15 August 2026

## Purpose

This document establishes an initial workload estimate for comparing cloud deployment options and pricing plans. It is a capacity-planning model rather than a measured production baseline. Assumptions should be replaced with observed metrics after production service is restored.

The estimate focuses on the Eric's Barbers system, but the calculation method can be reused for future projects.

## Accepted Platform Direction

Railway has been selected as the managed host for the NestJS API, PostgreSQL, and future backend cron or worker services. Vercel remains the Next.js frontend and browser-BFF host. The decision and its alternatives are recorded in [[ADR 0026 - Use Railway For Backend Hosting]].

The target includes isolated Railway production and test environments. Production API and database services remain continuously available. The test API may use Railway Serverless sleeping after background traffic and database-connection behavior have been made compatible; the test database is initially treated as a persistent cost. Actual Railway usage must replace the estimates in this document after deployment.

## Baseline User Assumptions

| Measure | Initial assumption |
| --- | ---: |
| Total registered users | 100 |
| Daily active users | 30 |
| Daily-active-user rate | 30% |
| Business API requests per active user per day | 5 |
| Average days per planning month | 30 |

Total registered users primarily affects durable storage. Daily active users and their behaviour drive request volume.

## User-Driven API Requests

The baseline calculation is:

```text
30 daily active users × 5 API requests = 150 API requests per day
```

| Period | Estimated API requests |
| --- | ---: |
| Day | 150 |
| 30-day month | 4,500 |
| Year | 54,750 |

This is an extremely small request workload for a modern application platform. Request-count allowances are unlikely to constrain the initial deployment.

### Growth scenarios

| Scenario | Daily active users | Requests per active user | Requests/day | Requests/30-day month |
| --- | ---: | ---: | ---: | ---: |
| Initial | 30 | 5 | 150 | 4,500 |
| Five-times activity | 150 | 5 | 750 | 22,500 |
| Ten-times activity | 300 | 5 | 1,500 | 45,000 |
| Ten-times users and behaviour | 300 | 10 | 3,000 | 90,000 |

Even the ten-times scenarios remain modest. Cost decisions should therefore account for minimum instance charges and idle resource consumption rather than concentrating only on per-request pricing.

## Request-Path Accounting

One customer action does not always equal one billed request across the whole system.

### Browser request through the Next.js BFF

```text
Browser -> Next.js/BFF -> NestJS API -> PostgreSQL
```

One action can produce:

- one incoming request handled by Next.js;
- one outgoing request from Next.js;
- one incoming request handled by NestJS; and
- one or more PostgreSQL operations.

If all 150 daily actions pass through the BFF, NestJS still receives 150 API requests, but the two application runtimes collectively handle approximately 300 HTTP request legs per day. If Vercel and the API use different providers, the BFF-to-API traffic crosses provider boundaries.

### Mobile request

```text
React Native app -> NestJS API -> PostgreSQL
```

The mobile application calls NestJS directly, so one mobile action normally produces one incoming API request rather than a BFF request plus an API request.

Static assets, browser navigation, image requests, analytics, source maps, universal-link files, and mobile build downloads are not included in the 150 business API requests.

## Peak Traffic and Concurrency

Monthly request totals do not describe short bursts. Booking traffic may cluster around evenings, weekends, reminder emails, or newly released availability.

For an initial planning envelope, assume that 20% of daily requests could occur within the busiest ten minutes:

```text
150 × 20% = 30 requests in ten minutes
30 / 600 seconds = 0.05 average requests per second
```

The platform should nevertheless tolerate short bursts of at least 1–2 requests per second while preserving booking concurrency rules. This is not a measured requirement and should be verified with a small load test before production acceptance.

## Non-User Workload

### Email outbox polling

The NestJS email outbox processor currently runs once per minute, whether or not an email is waiting:

| Period | Scheduled outbox polls |
| --- | ---: |
| Day | 1,440 |
| 30-day month | 43,200 |
| Year | 525,600 |

Each run queries PostgreSQL. At the initial user estimate, this scheduled database activity is almost ten times the number of user-driven API requests.

The polling frequency is operationally reasonable for prompt email delivery, but it must be included when comparing database compute, autosuspend, and scale-to-zero plans. A database that wakes for every poll will not remain suspended.

### Cleanup jobs

The API currently schedules two daily authentication-cleanup jobs:

- unverified-user cleanup at 02:00; and
- expired authentication-state cleanup at 03:00.

Together, these add approximately 60 scheduled executions per 30-day month. Their request count is negligible, but their presence requires a runtime or external scheduler that reliably executes background work.

### Health and uptime checks

Health checks may exceed customer traffic:

| Check interval | Checks/day | Checks/30-day month |
| --- | ---: | ---: |
| 30 seconds | 2,880 | 86,400 |
| 1 minute | 1,440 | 43,200 |
| 5 minutes | 288 | 8,640 |

The current `/health` endpoint checks PostgreSQL and sends a real test email through Resend on every invocation. It is therefore unsuitable for frequent platform liveness or external uptime monitoring in its current form. Before production monitoring is configured, health responsibilities should be separated:

- a cheap liveness endpoint that confirms the process is running;
- a readiness endpoint that checks required dependencies without causing side effects; and
- a less frequent synthetic email-delivery check, if required.

At a five-minute interval, a side-effect-free external health check would add 8,640 HTTP requests per month, bringing the initial API HTTP total to approximately 13,140 requests per month.

### Retries and operational overhead

The estimate must allow for:

- safe client retries following timeouts;
- idempotent booking replays;
- webhook retries from external providers;
- accidental loops or misconfigured clients.

Bot, crawler, denial-of-service, credential-stuffing, and other deliberately abusive traffic are excluded from this initial usage estimate. The system should still retain normal security controls, but speculative hostile traffic will not be used to size the initial hosting plan.

Organic growth is represented separately by the growth scenarios above. Spend alerts and hard cost controls remain desirable safeguards against configuration mistakes and unexpected resource consumption.

## Database Usage

API request count is not equal to database-query count. A single booking or authentication request can perform validation reads, transactional writes, session updates, idempotency operations, and outbox writes.

Until query metrics are available, use a rough planning range of 5–15 database operations per business API request:

| Workload | Monthly database operations |
| --- | ---: |
| 4,500 user-driven requests at 5 operations each | 22,500 |
| 4,500 user-driven requests at 15 operations each | 67,500 |
| Email outbox polling | at least 43,200 |
| Indicative initial total | approximately 65,700–110,700 |

This range is only a sizing proxy and must not be treated as a billing total without checking how a provider meters database activity.

### Connections and pooling

Provider selection must account for:

- Prisma's connection pool per API process;
- additional connections during deployments and migrations;
- multiple API replicas;
- staging and preview environments;
- serverless functions creating concurrent connections; and
- provider connection limits or pooler availability.

An application with low request volume can still exhaust a small database's connection allowance if each runtime or preview environment opens its own pool.

### Storage

One hundred users require negligible raw storage. The larger contributors will be:

- bookings and historical snapshots;
- sessions, MFA challenges, password-reset and verification state;
- idempotency records;
- availability rules and exceptions;
- outbox events and retry history;
- PostgreSQL indexes and write-ahead logs;
- backups and point-in-time-recovery history; and
- application and audit logs stored outside PostgreSQL.

A 1–5 GB initial database allowance should provide ample application-data headroom, but backup retention, write-ahead logs, and provider minimum storage allocations must be priced separately. Data growth should be measured monthly.

## Network Transfer

Request count alone does not determine bandwidth. Until measured, use explicit planning assumptions rather than pretending the payload size is known.

For example, with an average 25 KB API response and 5 KB request body:

| Transfer | Monthly estimate |
| --- | ---: |
| API response data | 4,500 × 25 KB = approximately 112.5 MB |
| API request bodies | 4,500 × 5 KB = approximately 22.5 MB |

This excludes protocol overhead, health checks, static assets, images, deployment artifacts, source maps, logs, database backups, mobile releases, and cross-provider traffic. The initial business API bandwidth should still be comfortably below 1 GB per month.

If Next.js, NestJS, and PostgreSQL use different providers, check whether each provider charges for:

- public egress;
- cross-region transfer;
- traffic to an external managed database;
- image and build-artifact transfer; and
- backup export.

Keep the UI, API, and primary database in geographically close regions where practical.

## Compute and Memory

For an always-on NestJS process, idle memory and minimum instance size are likely to cost more than request execution.

The following must be measured before selecting a final plan:

- NestJS resident memory after startup;
- Next.js server memory after startup;
- memory during builds and Prisma generation;
- CPU and response time for availability calculations;
- cold-start behaviour where services can sleep;
- database compute consumed by minute-by-minute outbox polling; and
- resource contention when several projects share one VPS.

Measure the applications under representative traffic rather than selecting a plan solely from the 4,500-request monthly total.

## Environment Multiplier

Deployment cost is multiplied by the number of active environments:

| Environment | Expected runtime model |
| --- | --- |
| Local development | Developer machine and local PostgreSQL; no permanent cloud compute required. |
| Pull-request preview | Frontend preview by default; API/database previews only where the change requires them and preferably ephemeral. |
| Shared test or staging | May sleep or run on a schedule when not actively used. Must not share production credentials or customer data. |
| Production | Always available according to the accepted service objective. |

Running production-sized UI, API, worker, and database services continuously in both staging and production can approximately double the baseline cost. The initial strategy should use ephemeral previews and a low-cost or scheduled staging environment unless continuous staging is justified.

## Other Pricing Factors

Provider comparison must include more than request limits:

- minimum monthly plan or per-service charges;
- idle RAM and CPU billing;
- sleep, autosuspend, wake-up latency, and whether scheduled polling prevents suspension;
- build minutes, build concurrency, cache and artifact retention;
- production and preview deployment limits;
- database compute, storage, connection limits and pooling;
- automated backups, retention, point-in-time recovery and restore testing;
- outbound and cross-provider network transfer;
- log, metric, trace, error-report and audit retention;
- custom domains, TLS, DNS and static outbound IP charges;
- secrets and environment management;
- cron jobs, always-on workers and queues;
- availability targets, support and incident response;
- spend alerts, budgets and hard limits;
- data region, export and provider portability;
- development, staging and production duplication; and
- engineering time required to patch and operate self-hosted infrastructure.

## Portfolio-Level Planning

Future planning expects approximately 10–20 deployed services across multiple projects. The workloads will not all have the same criticality.

The provider evaluation should separate:

- client production services that need durable paid infrastructure;
- low-traffic APIs that can share a VPS safely;
- static or serverless frontends with minimal idle cost;
- development and demonstration services that may sleep;
- databases requiring project-level isolation; and
- shared platform services such as monitoring, deployment management and log collection.

At portfolio scale, a fixed-price VPS can reduce per-service compute cost, while managed PostgreSQL can preserve database durability. The trade-off is a larger shared failure domain and more operational responsibility.

## Accepted Capacity Position

The initial workload does not require high compute capacity or request-scale infrastructure. The Railway implementation should prioritise:

1. durable PostgreSQL with backups and a documented restore path;
2. enough memory for the always-on NestJS process and its scheduled jobs;
3. predictable minimum monthly cost;
4. straightforward deployments, migrations and rollback;
5. spend controls and useful logs;
6. a path to host additional small services economically; and
7. the ability to scale an individual successful project independently later.

## Open Inputs

The following inputs are still needed to finalise production operations and revise the estimate:

- acceptable monthly budget for the current project and the wider portfolio;
- target production availability and acceptable maintenance downtime;
- expected number of bookings, logins, MFA attempts and emails per day;
- expected split between browser and mobile users;
- measured average and high-percentile API response size;
- measured idle and peak memory for Next.js and NestJS;
- desired backup frequency, retention and recovery-point objective;
- desired recovery-time objective;
- whether measured test-database cost justifies a later scale-to-zero alternative;
- whether different client projects may share infrastructure; and
- data residency or contractual requirements.

## Revision Process

After deployment is restored, record measured values for at least 30 days and revise this estimate. At minimum, capture daily requests, active users, response sizes, CPU, memory, database size, connections, slow queries, email volume, error rate, log volume and backup size.
