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
| [[ADR 0005 - Use JWT Access Tokens With Refresh Token Sessions]] | Use access tokens plus database-backed refresh-token sessions. | Accepted |
| [[ADR 0006 - Use Next.js API Routes For Frontend Auth Cookies]] | Use Next.js route handlers to manage frontend auth cookies. | Accepted |
| [[ADR 0007 - Require Email Verification Before Login]] | Require users to verify email before logging in. | Accepted |
| [[ADR 0008 - Use Generated OpenAPI Client For Frontend API Calls]] | Generate frontend API client code from OpenAPI. | Accepted |
| [[ADR 0009 - Use Feature Flags For Booking Availability]] | Gate incomplete booking functionality behind feature flags. | Accepted |
| [[ADR 0010 - Use One Next.js App For Customer And Staff Domains]] | Serve customer and staff subdomains from one host-aware Next.js app. | Accepted |
| [[ADR 0011 - Use Role-Aware Login Redirects In The Next.js BFF]] | Decode the access token in the BFF to choose post-login destinations. | Accepted |
| [[ADR 0012 - Keep The NestJS API On An Always-On Backend Host]] | Keep the NestJS API on an always-on backend host instead of Vercel Functions. | Accepted |
| [[ADR 0013 - Model Services As Database Records]] | Store services as business data with price, duration, description, and lifecycle state. | Accepted |
| [[ADR 0014 - Model Barber Availability With Rules And Exceptions]] | Compute barber slots from weekly rules, date exceptions, and existing bookings. | Accepted |
| [[ADR 0015 - Use Transactional Outbox For Operational Emails]] | Record email work in an outbox table and process it asynchronously. | Accepted |
| [[ADR 0016 - Enforce Booking Integrity In The API And Database]] | Combine API validation with database constraints for booking correctness. | Accepted |
| [[ADR 0017 - Use UUID Booking References As Bearer Credentials]] | Use high-entropy booking UUIDs for guest booking management by reference. | Accepted |
| [[ADR 0018 - Scope Booking Access By Role And Reference]] | Apply booking data scoping by role in the backend service layer. | Accepted |
| [[ADR 0019 - Use Email MFA And Feature-Flag External Providers]] | Use email-code MFA now and keep external identity providers behind a feature flag. | Accepted |
| [[ADR 0020 - Use Soft Delete And Anonymization For Account Deletion]] | Soft-delete and anonymize accounts by default while preserving operational reporting history. | Accepted |
| [[ADR 0021 - Allow Public Customer Booking Entry Points]] | Let unauthenticated users view customer booking entry pages while keeping staff/account booking access protected. | Accepted |
| [[ADR 0022 - Build A Customer Mobile App With React Native And Expo]] | Build a customer-only React Native and Expo app that communicates directly with NestJS while preserving guest booking. | Accepted |
| [[ADR 0023 - Snapshot Accepted Service Terms On Bookings]] | Preserve the service name, duration, and price accepted for each booking independently of later catalogue changes. | Accepted |
| [[ADR 0024 - Make Booking Creation Idempotent]] | Use client-generated idempotency keys and database-backed replay semantics for booking creation. | Accepted |
| [[ADR 0025 - Establish The NestJS OpenAPI Document As The Canonical Client Contract]] | Generate and commit one deterministic API-owned OpenAPI document for web and mobile clients. | Accepted |

## Suggested Future Records

Potential ADRs to add later:

- payment provider choice
- testing strategy
