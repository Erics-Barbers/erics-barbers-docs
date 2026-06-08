# ADR 0003 - Use Prisma And PostgreSQL For Persistence

Status: Accepted

Date: 2026-05-10

## Context

The application needs durable relational data:

- users
- sessions
- bookings
- barbers
- external accounts
- MFA settings

The data has clear relationships. For example, a user can have many bookings, a user can have many sessions, and a barber is linked to a user account.

## Decision

Use PostgreSQL as the database and Prisma as the ORM.

The schema is defined in:

`erics-barber-api/prisma/schema.prisma`

## Alternatives Considered

## Raw SQL

Pros:

- full control over database queries
- no ORM abstraction
- excellent for complex SQL

Cons:

- more manual query writing
- more manual type mapping
- more boilerplate for common CRUD operations
- higher chance of query/string mistakes for a solo project

## TypeORM

Pros:

- common in NestJS projects
- decorator-based entity modelling
- mature ORM ecosystem

Cons:

- entity classes can become coupled to persistence details
- Prisma's generated client and migration workflow felt more direct for this project

## MongoDB

Pros:

- flexible document model
- fast to iterate for unstructured data

Cons:

- the project data is naturally relational
- bookings, users, barbers, and sessions need clear relationships
- relational constraints are useful for this domain

## Decision Rationale

PostgreSQL was chosen because the domain is relational.

Prisma was chosen because it provides:

- a schema-first way to model data
- generated TypeScript types
- migrations
- a readable query API
- good developer experience
- easy integration with PostgreSQL

## Trade-Offs

Prisma makes common database work easier, but it also couples parts of the application to Prisma-generated types.

In the current backend, some use cases and services import generated Prisma model types. This is convenient, but it means the application layer is not fully independent from the persistence layer.

## Consequences

Positive consequences:

- database schema is easy to inspect
- generated types improve safety
- migrations keep database changes tracked
- repository code is easier to read than raw SQL for common operations

Negative consequences:

- Prisma-generated files exist inside the source tree
- application code can become coupled to Prisma types
- advanced database behavior may require dropping closer to raw SQL or careful Prisma modelling

## Current Follow-Up Work

- decide how strongly to isolate application use cases from Prisma types
- model services properly before completing booking flow
- add booking status and availability concepts when needed
- review session and refresh-token storage model

