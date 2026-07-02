# ADR 0012 - Keep The NestJS API On An Always-On Backend Host

Status: Accepted

Date: 2026-07-01

## Context

The frontend is deployed on Vercel.

The backend is a NestJS API that supports authentication, sessions, bookings, barbers, health checks, and future notification workflows.

The project may need background or scheduled work such as:

- cleaning expired sessions and MFA challenges
- sending appointment reminders
- sending booking confirmation or cancellation notifications
- running notification retry workers

Vercel can host backend code through functions, but that model is request-driven and may experience cold starts. It is not equivalent to a traditional always-running web service.

## Decision

Keep the NestJS API on an always-on backend hosting platform rather than migrating it to Vercel Functions.

Vercel remains the frontend/BFF hosting platform:

```text
Next.js frontend and BFF routes -> Vercel
NestJS API                      -> always-on backend host, such as Render
PostgreSQL                      -> managed database
```

## Alternatives Considered

## Host NestJS On Vercel Functions

Pros:

- fewer hosting providers
- shared deployment platform with the frontend
- branch/preview deployment workflow could be simpler
- Vercel supports backend frameworks and function-based compute

Cons:

- not an always-running service model
- cold starts can affect API response time
- long-running background workers do not fit naturally
- persistent in-process state cannot be relied on
- scheduled work would need to be triggered as HTTP endpoints or external jobs
- database connection pooling becomes more important

## Self-Host On A VPS

Pros:

- full control over the server
- always-on processes
- one place for Next.js, NestJS, workers, and reverse proxying
- useful infrastructure learning

Cons:

- the developer owns server updates, TLS, firewalls, deployments, monitoring, backups, and incident response
- more operational work for a solo developer
- easier to misconfigure security-sensitive infrastructure

## Use Only Next.js API Routes

Pros:

- one application and one deployment target
- simpler for a small purely frontend-adjacent backend

Cons:

- weak fit for a structured NestJS API
- mixes domain workflows into the frontend project
- less natural for future workers, queues, and background notifications

## Decision Rationale

The NestJS API is expected to behave like a traditional backend service.

An always-on host is a better fit because:

- the API should be warm and responsive for UI requests
- future background notification work is likely
- scheduled cleanup jobs already exist in the backend design
- the backend can keep a normal service/process model
- frontend and API responsibilities remain separated

## Trade-Offs

The project uses more than one hosting platform.

That means:

- more environment variables to coordinate
- more deployment dashboards
- CORS and API base URLs must be configured carefully
- observability spans multiple providers

The trade-off is acceptable because it preserves a better runtime model for the backend.

## Consequences

Positive consequences:

- NestJS can run as an always-on API
- future background workers and notification processes have a natural home
- Vercel can remain focused on the Next.js frontend and BFF routes
- cold-start risk is reduced for core API calls

Negative consequences:

- deployments are split across Vercel and the backend host
- preview environments may need more coordination
- local and production environment documentation must stay accurate

## Follow-Up Work

- document production deployment and environment variables
- decide how notification jobs will run: scheduled API job, dedicated worker, or queue consumer
- define monitoring and alerting for both Vercel and the backend host
- create a deployment checklist before production launch
