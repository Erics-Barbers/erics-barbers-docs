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

## Render Retirement

The historical Render API and PostgreSQL deployment is retired as a target architecture. The old Render database does not need to be recovered for the current restoration work.

Render-specific URLs, deployment assumptions, and configuration should be treated as historical evidence only. New deployment work targets Railway for backend services and Vercel for the web frontend and BFF.

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

This verifies that the API process can reach PostgreSQL without sending email.

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

Remaining production observability decisions:

- choose and configure application-level error monitoring;
- decide whether production error alerts should be delivered by email;
- define alert thresholds and recipients without recording secret values;
- record uptime alert configuration for production `/health/ready`; and
- record backup, export, and restore evidence.

## Open Verification

- Verify production and test use isolated Railway PostgreSQL databases and environment-scoped credentials.
- Verify production API uses the private Railway database URL.
- Confirm test API sleeping behaviour is acceptable with automated health checks disabled.
- Verify Railway backup, export, restore, retention, and recovery behaviour.
- Record deployment, rollback, and restore evidence before closing the platform tickets.
