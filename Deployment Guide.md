# Deployment Guide

Status: in progress

This guide records the Railway and Vercel deployment process for Eric's Barbers. It intentionally records operational facts, commands, and verification evidence without storing secret values.

## Current Deployment Model

Production:

```text
Vercel production deployment
└── Next.js frontend and browser BFF

Railway production environment
├── NestJS API
└── PostgreSQL
```

Test:

```text
Vercel test deployment
└── Next.js frontend and browser BFF

Railway test environment
├── NestJS API
└── isolated PostgreSQL
```

The production API remains continuously available. The test API is allowed to sleep, so no automated test health check is configured by default. Manual test health checks can still be called when needed.

## Legacy Deployment Retirement

The historical API and PostgreSQL deployment is retired as a target architecture. The old database does not need to be recovered for the current restoration work.

Legacy provider URLs, deployment assumptions, and configuration should be treated as historical evidence only. New deployment work targets Railway for backend services and Vercel for the web frontend and BFF.

## API Build And Deployment Commands

Railway API build command:

```bash
npm run build:deploy
```

This generates the Prisma client and builds the NestJS application.

Railway API pre-deploy command:

```bash
npm run db:deploy
```

This applies committed Prisma migrations to the Railway PostgreSQL database. It must run in the pre-deploy phase rather than the image-build phase because Railway private database networking is available at runtime, not during image build.

Railway API start command:

```bash
npm run start:prod
```

## Health Checks

Production health check:

```text
GET /health/ready
```

This verifies that the API process can reach PostgreSQL and, when configured,
that the Sentry SDK is initialized. Production readiness fails when
`SENTRY_ENVIRONMENT=production` but `SENTRY_DSN` is missing. Outside production,
an omitted DSN is treated as Sentry being intentionally disabled. The check does
not send email or emit a synthetic Sentry event.

Test health checks are not automated while the test API is intended to sleep. Manual checks remain available:

```text
GET /health/live
GET /health/ready
GET /health/email
```

`/health/email` sends a real synthetic email to `HEALTH_CHECK_EMAIL_TO` and should not be used as a frequent uptime probe.

## Background Jobs

Production should run scheduled background work unless a later operational decision moves it into separate Railway cron or worker services:

```text
EMAIL_OUTBOX_PROCESSOR_ENABLED=true
AUTH_CLEANUP_JOBS_ENABLED=true
```

Test may disable scheduled work to support sleeping and cost control:

```text
EMAIL_OUTBOX_PROCESSOR_ENABLED=false
AUTH_CLEANUP_JOBS_ENABLED=false
```

The email outbox sends queued auth and booking emails. The auth cleanup jobs delete stale unverified users, expired refresh sessions, and expired MFA challenges.

## Observability

Current confirmed observability:

- Vercel deployment logs are visible.
- Railway deployment and runtime logs are visible.
- Railway service and database metrics are visible.
- production `/health/ready` is configured for the API health check.
- Sentry monitors production API and web exceptions and sends email alerts to the production alert mailbox.
- Vercel and Railway deployment notifications are enabled and verified.
- Sentry source maps have been verified against production web errors, including original TypeScript stack frames.
- Sentry continuously monitors `https://api.erics-barbers-luton.co.uk/health/ready` and sends failure and recovery notifications.
- Automated uptime monitoring remains disabled for test so that the test API can sleep.

Sentry configuration values:

- API on Railway: `SENTRY_DSN`, `SENTRY_ENVIRONMENT`, `SENTRY_RELEASE`, and optionally `SENTRY_TRACES_SAMPLE_RATE`.
- Web on Vercel: `SENTRY_DSN`, `NEXT_PUBLIC_SENTRY_DSN`, `SENTRY_ENVIRONMENT`, `NEXT_PUBLIC_SENTRY_ENVIRONMENT`, `SENTRY_RELEASE`, `NEXT_PUBLIC_SENTRY_RELEASE`, and optionally `SENTRY_TRACES_SAMPLE_RATE` / `NEXT_PUBLIC_SENTRY_TRACES_SAMPLE_RATE`.
- Web source-map uploads on Vercel: `SENTRY_ORG`, `SENTRY_PROJECT`, and `SENTRY_AUTH_TOKEN`.

Do not commit DSNs, auth tokens, or alert-recipient email addresses to the repositories.

Temporary web diagnostics can be enabled in Vercel with
`SENTRY_DEBUG_ENABLED=true`. Visit `/debug-sentry` to trigger separate browser
and Next.js server errors, then confirm both events and their source-mapped stack
traces in Sentry. Remove the variable after verification; the page and its API
route return `404` while the flag is disabled.

## Verification Status

Completed:

- Production and test Railway API deployments build and deploy from Git.
- Production and test PostgreSQL services, variables, credentials, and data are isolated.
- API services use their environment-specific private Railway database connection.
- Production remains continuously available.
- Test API sleeping behaviour is accepted by leaving automated test health checks disabled.
- Test data is synthetic and cannot mutate production services.
- Production readiness checks use side-effect-free `/health/ready`.
- Background-job behaviour is explicitly configured per environment.
- Prisma migrations run through the Railway pre-deploy command.
- R2 database backups can be created from the Railway backup service.
- A backup can be restored from R2 through the manual restore service.
- Vercel, Railway API, and Railway database logs or metrics are visible.
- Sentry SDK wiring, production alert routing, and web source maps are verified.
- Vercel and Railway deployment-failure notifications are verified.
- Production uptime failure and recovery notifications are verified against the custom API domain.

## Open Verification

No deployment verification items remain open.
