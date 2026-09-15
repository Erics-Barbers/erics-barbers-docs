# ADR 0026 - Use Railway For Backend Hosting

Status: Accepted

Date: 2026-09-15

## Context

The Eric's Barbers system needs a sustainable host for its shared backend services and PostgreSQL database. The previous arrangement placed the Next.js frontend and browser BFF on Vercel, the NestJS API on Render, and PostgreSQL on Render. The expiry of the Render PostgreSQL free plan stopped the database and made the production application unavailable.

The expected workload is small: approximately 100 registered users, 30 daily active users, and 4,500 user-driven API requests per month. Request throughput is therefore not the main constraint. The more important concerns are database durability, predictable minimum cost, operational simplicity, observability, repeatable deployments, and support for scheduled or background work.

The project is maintained by a solo developer who expects to have limited operational time while working in a full-time role. The selected platform should reduce routine server administration and provide a simple Git-based development experience.

The delivery workflow also needs an isolated backend test environment with a stable API address for the web test deployment and mobile preview builds. Test must not use production credentials or customer data. It may trade first-request latency for lower idle cost.

## Decision

Use Railway as the managed hosting platform for backend services in both production and test environments.

The target deployment shape is:

```text
Production
├── Vercel
│   └── Next.js frontend and browser BFF
└── Railway production environment
    ├── NestJS API
    ├── PostgreSQL
    └── future backend cron jobs or workers

Test
├── Vercel test deployment
└── Railway test environment
    ├── NestJS API
    ├── isolated PostgreSQL
    └── test-only jobs where required

Mobile
├── preview build -> Railway test API
└── production build -> Railway production API
```

The production NestJS API remains continuously available, preserving the runtime model selected in [[ADR 0012 - Keep The NestJS API On An Always-On Backend Host]]. The production PostgreSQL database is a separate Railway service and uses persistent storage.

The Railway test environment uses separate variables, secrets, networking, database storage, and synthetic data. The test API may use Railway Serverless mode when its background polling, database connections, and telemetry have been made compatible with sleeping. The exact test-service lifecycle will be verified through implementation and cost measurement; the production API and databases must not depend on application sleeping.

Vercel remains the host for the Next.js frontend and BFF. The React Native application is distributed through the Apple App Store and Google Play and calls the Railway API directly.

## Decision Rationale

Railway provides the best current balance for this project:

- managed application runtime and PostgreSQL reduce operational responsibility;
- Git-based builds and deployments keep the developer workflow simple;
- production and persistent test environments can be isolated within one project model;
- private networking keeps API-to-database traffic off the public internet within each environment;
- built-in deployment logs and CPU, memory, disk, and network metrics provide a useful infrastructure baseline;
- usage-based billing is appropriate for small services, while resource and usage limits can bound unexpected cost;
- serverless sleeping can reduce idle application compute in test where the workload is compatible;
- ordinary Node.js and PostgreSQL keep the application portable if the platform is changed later; and
- Railway naturally accommodates future cron services, workers, and notification processing without converting NestJS to Vercel Functions.

Keeping all backend services on one managed platform is preferred to splitting the API and database across providers because it reduces dashboards, credentials, network paths, latency variables, and incident diagnosis work.

## Alternatives Considered

### Continue With Render

Advantages:

- the API has previously been deployed there;
- managed Node.js and PostgreSQL are available; and
- Render provides logs, metrics, health checks, and infrastructure-as-code.

Reasons not selected:

- expiry of the previous free database already caused a production outage;
- restoring the existing arrangement would not improve confidence in the project's cost and service-lifecycle model; and
- Railway's environment and usage model is a better fit for the preferred production-plus-test workflow.

### Railway API With Neon PostgreSQL

Advantages:

- database branching and scale-to-zero test compute;
- specialist managed PostgreSQL features; and
- a useful fallback if Railway database recovery proves insufficient.

Reasons not selected initially:

- adds another provider, billing surface, credential set, and failure boundary;
- removes Railway private networking between the API and database; and
- increases operational and diagnostic complexity for a very small workload.

This remains the preferred fallback if Railway's verified backup and recovery capabilities do not satisfy the production recovery requirements.

### Shared VPS With Containers

Advantages:

- potentially lower compute cost across a future portfolio of many small services;
- full control over runtime and networking; and
- efficient sharing of fixed resources.

Reasons not selected:

- the developer would own operating-system and container-runtime updates, TLS, firewalling, orchestration, monitoring, backups, capacity, and incident recovery;
- several applications would share a larger failure domain; and
- the operational burden conflicts with the requirement to minimise maintenance during full-time employment.

A VPS can be reconsidered for non-critical services when the portfolio is large enough to justify a separate operational platform.

### Host NestJS On Vercel Functions

Advantages:

- one application provider; and
- potentially simpler frontend preview integration.

Reasons not selected:

- the API contains scheduled cleanup and email-outbox processing;
- the established NestJS process and Prisma connection model fit a conventional managed service better; and
- it would revisit the runtime decision already accepted in ADR 0012 without a compelling benefit.

## Trade-Offs

### Positive

- substantially less operational responsibility than a VPS;
- one provider for API, database, and future backend workers;
- isolated production and test backend environments;
- simple Git-connected deployments and centralized backend logs;
- private service networking and low application/database latency;
- a clear place for future scheduled and background work; and
- a portable Node.js and PostgreSQL architecture.

### Negative

- the system still spans Vercel, Railway, Expo/app stores, and external services such as Resend;
- Railway usage billing is less fixed than a prepaid VPS and needs alerts and limits;
- an idle persistent test database consumes resources even when the test API sleeps;
- Railway Serverless wake-up may delay or fail the first test request and is unsuitable for the production API;
- minute-by-minute outbox polling, open database connections, or telemetry may prevent the test API from sleeping;
- placing API and database together increases exposure to a Railway platform or regional incident;
- lower-cost community support may not be sufficient if the service becomes business-critical; and
- platform-specific deployment configuration creates some vendor dependency even though the application and database remain portable.

## Consequences

- New backend deployment work targets Railway rather than Render.
- Render-specific URLs, startup assumptions, and deployment documentation must be removed or clearly marked historical.
- Railway production and test environments require separate domains, variables, credentials, and databases.
- Production and test must use private database URLs within their respective Railway environments.
- Production data must never be copied into test without an accepted anonymisation process.
- The API needs side-effect-free liveness and readiness endpoints before uptime monitoring is enabled.
- Test uptime monitoring must not continually wake sleeping services.
- Background polling and scheduled work must be configurable per environment or moved to explicit cron/worker services where appropriate.
- Platform metrics must be supplemented with application-level error reporting and tracing.
- Database backup, restore, export, retention, and recovery testing remain release gates; choosing Railway does not itself satisfy them.
- Cost and resource use must be measured after deployment and the initial estimates revised.

## Follow-Up Work

- formally retire the expired Render database; recovery is not required for the current restoration work;
- create Railway production and isolated test environments;
- configure production and test domains, secrets, and private database connections;
- define the database migration and application deployment order;
- verify Railway backup, restore, export, retention, and recovery behaviour;
- separate liveness, readiness, and synthetic email checks;
- make background jobs explicitly configurable by environment;
- test whether the test API can reliably enter and recover from Serverless sleep;
- add infrastructure limits, spend alerts, logs, application error monitoring, and uptime alerts;
- document deployment, rollback, database recovery, and Render cutover procedures; and
- measure CPU, memory, storage, connections, network transfer, cold-start behaviour, and monthly cost.

