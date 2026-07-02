# ADR 0008 - Use Generated OpenAPI Client For Frontend API Calls

Status: Accepted

Date: 2026-05-10

## Context

The backend exposes a Swagger/OpenAPI spec. The frontend needs to call backend endpoints for auth, bookings, barbers, and future features.

Writing every request manually can lead to duplicated URL strings, inconsistent request bodies, and weaker type safety.

## Decision

Use `openapi-typescript-codegen` to generate frontend API client code from the OpenAPI spec.

Generated client location:

`erics-barbers-ui/api/generated`

Repository wrapper location:

`erics-barbers-ui/api/repositories`

## Alternatives Considered

## Manual Fetch Calls Everywhere

Pros:

- simple and explicit
- no generation step
- easy to customize each request

Cons:

- repeated endpoint strings
- easy to drift from backend DTOs
- more boilerplate
- less type safety

## Shared TypeScript Types Package

Pros:

- frontend and backend can share DTO types
- strong compile-time consistency

Cons:

- requires package/workspace setup
- can tightly couple frontend and backend source code
- still does not generate request methods

## Use React Query Or SWR Without Generated Client

Pros:

- good frontend data-fetching patterns
- caching and loading states

Cons:

- still need request functions
- does not solve backend contract generation by itself

## Decision Rationale

The generated OpenAPI client gives the frontend a structured way to call backend endpoints and stay aligned with the API contract.

This is especially useful because NestJS can generate Swagger documentation from controllers and DTOs.

## Trade-Offs

The generated client adds a build step and can become stale if the OpenAPI spec is not updated.

It also has limitations around cookie-based auth. The current generated client has `WITH_CREDENTIALS` set to `false`, which means direct browser calls may not include cookies unless configured.

## Consequences

Positive consequences:

- less manual request code
- API methods are grouped by backend controller
- frontend can rely on generated model types
- useful as the API surface grows

Negative consequences:

- generated files should not be manually edited
- client must be regenerated after API changes
- cookie/auth behavior must be configured carefully
- the project currently mixes generated-client calls with Next.js route handlers

## Current Follow-Up Work

- update OpenAPI spec generation workflow
- use generated DTO/model types where useful
- use generated request methods for non-auth backend resources as those UI flows are built

## Status Update - 2026-07-02

Auth browser flows are intentionally excluded from direct generated-client usage.

Decision refinement:

- browser auth UI calls local Next.js BFF routes under `/api/auth/*`
- BFF route handlers own cookie setting, refresh retries, logout cleanup, and same-origin checks
- generated auth DTO/model types may still be useful
- generated auth request methods should not be used directly from browser auth flows
- non-auth API areas, such as services, bookings, barbers, and future admin operations, can use the generated client where it fits

This keeps the generated client useful without letting it bypass the BFF auth boundary.
