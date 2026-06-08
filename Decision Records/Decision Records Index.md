# Decision Records Index

This folder contains Architecture Decision Records for Eric's Barbers.

An ADR captures:

- the decision being made
- the context at the time
- alternatives that could have been chosen
- trade-offs
- consequences for future development

These records are not meant to be perfect or permanent. They document the reasoning behind the project as it exists today, so future developers can understand why the codebase looks the way it does.

## Records

| ADR | Decision | Status |
| --- | --- | --- |
| [[ADR 0001 - Use Next.js For The Frontend]] | Use Next.js as the main frontend framework. | Accepted |
| [[ADR 0002 - Use NestJS For The Backend API]] | Use NestJS for the backend API. | Accepted |
| [[ADR 0003 - Use Prisma And PostgreSQL For Persistence]] | Use Prisma ORM with PostgreSQL. | Accepted |
| [[ADR 0004 - Organize Backend Around Modules And Use Cases]] | Use a modular, use-case-oriented backend structure. | Accepted |
| [[ADR 0005 - Use JWT Access Tokens With Refresh Token Sessions]] | Use access tokens plus database-backed refresh-token sessions. | Accepted, needs refinement |
| [[ADR 0006 - Use Next.js API Routes For Frontend Auth Cookies]] | Use Next.js route handlers to manage frontend auth cookies. | Accepted, needs refinement |
| [[ADR 0007 - Require Email Verification Before Login]] | Require users to verify email before logging in. | Accepted |
| [[ADR 0008 - Use Generated OpenAPI Client For Frontend API Calls]] | Generate frontend API client code from OpenAPI. | Accepted, needs consolidation |
| [[ADR 0009 - Use Feature Flags For Booking Availability]] | Gate incomplete booking functionality behind feature flags. | Accepted |

## Suggested Future Records

Potential ADRs to add later:

- customer and barber experiences in one app versus separate apps or subdomains
- modelling services as an enum versus a database table
- booking status lifecycle
- barber availability model
- payment provider choice
- deployment platform choice
- testing strategy

