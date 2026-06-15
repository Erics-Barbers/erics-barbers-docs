# ADR 0002 - Use NestJS For The Backend API

Status: Accepted

Date: 2026-05-10

## Context

The backend needed to support authentication, bookings, barbers, sessions, email verification, health checks, and eventually payments or notifications.

The project was intended to be more than a small script or simple CRUD API. It needed a structure that could grow as new features were added.

## Decision

Use NestJS for the backend API.

The active backend project is:

`erics-barber-api`

## Alternatives Considered

## Express

Express would have been a lighter-weight option.

Pros:

- minimal framework overhead
- quick to start
- very flexible
- large ecosystem

Cons:

- project structure would need to be invented manually
- dependency injection, modules, validation, guards, and testing patterns would require more setup
- easier for a growing codebase to become inconsistent

## Fastify

Fastify could have provided a fast Node.js API foundation.

Pros:

- strong performance
- schema-driven validation support
- good plugin model

Cons:

- less opinionated structure than NestJS
- less obvious module/use-case organization out of the box
- not as aligned with the project's learning goal around enterprise-style TypeScript backends

## Next.js API Routes Only

The entire backend could have been implemented inside the Next.js app.

Pros:

- one project instead of two
- easier deployment for a small app
- frontend and API code colocated

Cons:

- backend domain logic could become mixed into the frontend project
- less clear separation between product UI and API services
- less suitable if the API later needs to serve multiple clients

## Decision Rationale

NestJS was chosen because it provides a structured backend architecture from the start.

Useful NestJS features for this project:

- modules for feature boundaries
- controllers for HTTP endpoints
- injectable services and use cases
- guards for authentication and authorization
- DTO validation with `class-validator`
- Swagger/OpenAPI integration
- testing utilities
- familiar enterprise-style patterns

## Trade-Offs

NestJS adds boilerplate. A feature often needs a module, controller, DTOs, use cases, and services. For a small application, that can feel heavier than necessary.

The benefit is that the structure becomes valuable as the app grows. Authentication, bookings, barbers, payments, notifications, and health checks can each live in clear module boundaries.

## Consequences

Positive consequences:

- clear backend structure
- easier to add feature modules
- controller/use-case/service separation is visible
- good foundation for tests and documentation

Negative consequences:

- more files per feature
- dependency injection can feel indirect at first
- some modules currently look like placeholders because the structure exists before all features are complete

## Current Follow-Up Work

- keep controllers thin
- implement missing role guard behavior
- avoid creating unnecessary layers for very small features

## Status Update - 2026-06-13

The API now uses a strict global `ValidationPipe` with DTO whitelisting, unknown-property rejection, transformation, and sanitized validation errors.

This means DTOs are treated as the public request contract. New request fields should be added with explicit validation decorators, otherwise they will be rejected by the global pipe.
