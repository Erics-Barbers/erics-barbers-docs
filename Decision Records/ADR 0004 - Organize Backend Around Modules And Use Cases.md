# ADR 0004 - Organize Backend Around Modules And Use Cases

Status: Accepted

Date: 2026-05-10

## Context

The backend needed to support multiple domains:

- authentication
- bookings
- barbers
- health checks
- notifications
- payments

Putting all logic directly in controllers or services would be simple at first, but harder to maintain as features grow.

## Decision

Organize backend features around NestJS modules and application use cases.

The common structure is:

```text
module/
  presentation/      controllers and DTOs
  application/       use cases
  infrastructure/    Prisma repositories and external services
  domain/            domain entities, where present
```

## Alternatives Considered

## Controller-Service CRUD Structure

Pros:

- simpler file structure
- less boilerplate
- common in small NestJS apps

Cons:

- services can become too large
- business workflows can become mixed with persistence details
- harder to see user-level actions like register, login, refresh, and verify email

## Fully Clean Architecture With Ports Everywhere

Pros:

- stronger dependency inversion
- easier to swap infrastructure
- highly testable application layer

Cons:

- more abstraction
- more setup
- potentially too heavy for the project's current size
- can slow progress while the product shape is still changing

## Decision Rationale

The chosen structure gives the project some clean architecture benefits without going fully abstract.

Use cases make major actions visible:

- `RegisterUseCase`
- `LoginUseCase`
- `VerifyEmailUseCase`
- `LogoutUseCase`
- `CreateBookingUseCase`

This makes the backend easier to explain and easier to test at the business-action level.

## Trade-Offs

The main trade-off is extra files. Some use cases are currently thin wrappers around repository methods, especially in the booking module.

That is acceptable if those workflows are expected to grow. If they do not grow, the extra indirection may be unnecessary.

## Consequences

Positive consequences:

- controllers stay focused on HTTP concerns
- business actions have clear names
- modules give the codebase clear boundaries
- future developers can find feature logic quickly

Negative consequences:

- more boilerplate
- some abstractions are not fully used yet
- the codebase is clean-architecture-inspired, but not fully dependency-inverted

## Current Follow-Up Work

- decide whether to fully use `application/ports` interfaces or remove unused ones
- keep use cases meaningful
- avoid adding use cases that only forward calls unless future logic is expected
- document module boundaries as booking and barber features mature

