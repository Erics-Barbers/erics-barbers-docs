# Database Design

This note explains the current database design for Eric's Barbers, based on the Prisma schema in the backend project:

`erics-barber-api/prisma/schema.prisma`

The database is PostgreSQL, accessed through Prisma. The schema is centred around users, authentication sessions, barbers, and bookings.

Related note: [[Roles and Permissions]]

## High-Level Overview

At a high level, the application stores:

- users who can register, log in, and make bookings
- barbers who are linked to user accounts
- bookings created by users and optionally assigned to barbers
- sessions for refresh-token based authentication
- external accounts for future social login support
- MFA records for future multi-factor authentication support

```mermaid
erDiagram
    User ||--o{ ExternalAccount : has
    User ||--o| Mfa : has
    User ||--o{ Session : creates
    User ||--o{ Booking : makes
    User ||--o| Barber : "may become"
    Barber ||--o{ Booking : receives
    Barber ||--o{ Session : "may have"
```

## Main Entities

## User

The `User` table is the central table in the schema. Every customer, barber, or admin is represented as a user.

Key fields:

| Field             | Purpose                                                           |
| ----------------- | ----------------------------------------------------------------- |
| `id`              | Primary key. Generated with `cuid()`.                             |
| `name`            | Display name for the user. Defaults to `Anonymous`.               |
| `email`           | Unique login identifier.                                          |
| `passwordHash`    | Hashed password. Optional to allow for external auth in future.   |
| `role`            | Controls whether the user is an `ADMIN`, `BARBER`, or `CUSTOMER`. |
| `isEmailVerified` | Tracks whether the user has verified their email address.         |
| `deletedAt`       | Set when the account has been soft-deleted.                       |
| `anonymizedAt`    | Set when direct personal data has been anonymized.                |
| `createdAt`       | When the user was created.                                        |
| `updatedAt`       | Automatically updated when the user changes.                      |

Design decision:

The app uses a single `User` table for all account types, with a `role` enum to distinguish permissions. This keeps authentication simple because every person logs in through the same account model.

The intended definitions for `CUSTOMER`, `BARBER`, and `ADMIN` are documented in [[Roles and Permissions]].

Trade-off:

Role-based modelling is simpler than having separate `Customer`, `Barber`, and `Admin` tables, but it means the application code must enforce permissions carefully. The schema says what role a user has, but the API must still check what each role is allowed to do.

## Barber

The `Barber` table represents users who can receive bookings.

Key fields:

| Field           | Purpose                                               |
| --------------- | ----------------------------------------------------- |
| `id`            | Primary key.                                          |
| `userId`        | Unique link back to the `User` table.                 |
| `displayName`   | Public-facing barber name. Must be unique.            |
| `isActive`      | Allows a barber to be disabled without deleting them. |
| `phone`         | Barber contact number. Must be unique.                |
| `createdAt`     | When the barber record was created.                   |
| `deactivatedAt` | When the barber profile was disabled.                 |

Relationship:

```mermaid
erDiagram
    User ||--o| Barber : "has optional barber profile"

    User {
        string id PK
        string email UK
        Role role
    }

    Barber {
        string id PK
        string userId UK, FK
        string displayName UK
        boolean isActive
        string phone UK
    }
```

Design decision:

A barber is not a completely separate kind of account. A barber is a `User` with an additional `Barber` profile.

Trade-off:

This avoids duplicating login data, but it creates a two-step concept: first a user exists, then that user may also have a barber profile. The API needs to keep the user's `role` and the existence of a `Barber` record consistent.

Deletion policy:

Barber accounts are deactivated and anonymized rather than hard-deleted. Their historical bookings remain linked to the barber profile so accountability and reporting can still show booking counts and earnings over periods such as a week.

## Barber Availability

Barber availability is modelled as reusable weekly rules plus one-off date exceptions.

`BarberAvailabilityRule` stores the normal working pattern for a barber. Each row describes one active time range on one day of the week.

Key availability rule fields:

| Field         | Purpose                                               |
| ------------- | ----------------------------------------------------- |
| `id`          | Primary key. Generated with `cuid()`.                 |
| `barberId`    | Barber this rule belongs to.                          |
| `dayOfWeek`   | Day this rule applies to, Monday through Sunday.      |
| `startMinute` | Start time as minutes from local midnight.            |
| `endMinute`   | End time as minutes from local midnight.              |
| `isActive`    | Allows rules to be disabled without deleting history. |
| `createdAt`   | When the rule was created.                            |
| `updatedAt`   | When the rule was last changed.                       |

`BarberAvailabilityException` stores one-off changes such as time off, holidays, or extra opening time.

Key availability exception fields:

| Field         | Purpose                                                              |
| ------------- | -------------------------------------------------------------------- |
| `id`          | Primary key. Generated with `cuid()`.                                |
| `barberId`    | Barber this exception belongs to.                                    |
| `date`        | Local business date affected by the exception.                       |
| `startMinute` | Optional start time. Null with `endMinute` means the whole day.      |
| `endMinute`   | Optional end time. Null with `startMinute` means the whole day.      |
| `type`        | Whether the exception makes the period `AVAILABLE` or `UNAVAILABLE`. |
| `reason`      | Optional operational note.                                           |
| `createdAt`   | When the exception was created.                                      |

Availability and booking times are constrained to 30-minute boundaries. MVP bookings are 30 minutes long, so valid appointment times start and end on the hour or half hour.

Customer booking views use a day-by-day availability query after the customer has selected a barber. The backend computes available slots from active weekly rules, one-off exceptions, and existing bookings, then returns both a flat slot list and hourly groups for the UI.

Current query endpoint:

- `GET /barbers/:barberId/availability/slots?date=YYYY-MM-DD&serviceId=...`

Booking creation and booking updates must validate against the same computed availability. The UI should treat available slots as display data only; the backend remains responsible for rejecting stale or unavailable times.

Customer-facing booking policy is enforced in the backend and mirrored by the UI date pickers:

- online bookings are available from tomorrow onwards; customers cannot book for today
- customers can book up to 1 calendar month in advance
- customers can reschedule or cancel online up to the day before the appointment
- same-day reschedules or cancellations require contacting the shop

## Booking

The `Booking` table stores appointments.

Key fields:

| Field               | Purpose                                                                                 |
| ------------------- | --------------------------------------------------------------------------------------- |
| `id`                | Primary key. Generated with `uuid()`.                                                   |
| `userId`            | Optional customer account that made the booking. Guest bookings leave this empty.       |
| `customerName`      | Customer name captured for guest bookings and booking snapshots.                        |
| `customerEmail`     | Customer email used for booking confirmation.                                           |
| `customerPhone`     | Customer phone number for operational contact.                                          |
| `barberId`          | Optional barber assigned to the booking.                                                |
| `serviceId`         | Optional service selected for the booking.                                              |
| `status`            | Booking lifecycle state: `PENDING`, `CONFIRMED`, or `CANCELLED`.                        |
| `startTime`         | Appointment start time.                                                                 |
| `endTime`           | Appointment end time.                                                                   |
| `cancelledAt`       | When the booking was cancelled, if applicable.                                          |
| `cancelledByUserId` | Optional user account that cancelled the booking. Guest cancellations leave this empty. |
| `createdAt`         | When the booking was created.                                                           |

Booking rows are operational records. They should remain available for reporting even when a customer deletes their account. Barber accountability reports should rely on booking facts such as `barberId`, time period, status, service, and price rather than customer names or emails.

Relationship:

```mermaid
erDiagram
    User |o--o{ Booking : makes
    Barber ||--o{ Booking : receives
    Service ||--o{ Booking : describes

    User {
        string id PK
        string email UK
        Role role
    }

    Barber {
        string id PK
        string userId FK
        string displayName UK
    }

    Booking {
        string id PK
        string userId FK "optional"
        string customerName
        string customerEmail
        string customerPhone
        string barberId FK "optional"
        string serviceId FK "optional"
        BookingStatus status
        datetime startTime
        datetime endTime
        datetime cancelledAt
        string cancelledByUserId
        datetime createdAt
    }

    Service {
        string id PK
        string name UK
        string description
        int pricePence
        int durationMinutes
        boolean isActive
    }
```

Design decision:

A booking may belong to a registered user, but it can also be created as a guest booking. Guest bookings require customer name, email, and phone. When a user verifies an account email, unowned guest bookings with a matching customer email are linked to that user. The barber relationship remains optional in the schema, but the current customer booking flow selects a barber before the slot is chosen.

Trade-off:

An optional barber makes booking creation more flexible, but it also means the application must decide what an unassigned booking means operationally. For example, the UI and admin workflows need to make it clear whether a booking is confirmed, pending assignment, or incomplete.

Booking status:

Bookings use a `BookingStatus` enum with `PENDING`, `CONFIRMED`, and `CANCELLED`. The customer booking flow currently creates `CONFIRMED` bookings because the selected barber and slot are validated before the row is written. Cancelled bookings should remain stored for reporting, but they should not block future availability.

The database enforces one active booking per barber slot with a PostgreSQL partial unique index on `Booking.barberId` and `Booking.startTime` where the status is not `CANCELLED`. Application-level availability checks still provide friendly validation, but the database index is the final guard against concurrent double-booking.

Status changes should be restrained. General booking updates should only change rescheduling fields such as service, barber, and appointment time. Customer cancellation should go through the dedicated cancel endpoint, which sets the booking status to `CANCELLED`, records cancellation metadata, and rejects already-cancelled bookings after verifying the requester can access the booking. Online reschedule and cancellation requests are also rejected once the booking's shop-local date is today or in the past.

Service model:

Services are stored in a `Service` table rather than a Prisma enum. This lets the product attach operational details to each service without another schema redesign.

Key service fields:

| Field             | Purpose                                                                 |
| ----------------- | ----------------------------------------------------------------------- |
| `id`              | Primary key. Generated with `cuid()`.                                   |
| `name`            | Public service name. Must be unique.                                    |
| `description`     | Customer-facing service description.                                    |
| `pricePence`      | Service price stored in pence.                                          |
| `durationMinutes` | Appointment duration used to derive end time.                           |
| `isActive`        | Allows services to be hidden without deleting historical booking links. |
| `createdAt`       | When the service was created.                                           |
| `updatedAt`       | When the service was last changed.                                      |

Bookings reference services through `Booking.serviceId`. The field is nullable so historical rows and migration paths can remain valid, but new booking creation should require an active service.

## OutboxEvent

The `OutboxEvent` table stores reliable background work created by business transactions.

Booking create, update, and cancel operations write booking email events to this table in the same database transaction as the booking mutation. Auth flows also use this table for verification, password reset, and MFA code emails. A scheduled NestJS processor reads due events, sends the email through Resend, marks successful events as `PROCESSED`, and marks failed events as `FAILED` with retry metadata.

Key fields:

| Field         | Purpose                                                                                                                                                           |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`          | Primary key. Generated with `cuid()`.                                                                                                                             |
| `type`        | Event type, such as `BOOKING_CONFIRMATION_EMAIL` or `AUTH_VERIFICATION_EMAIL`.                                                                                    |
| `status`      | Processing state: `PENDING`, `PROCESSING`, `PROCESSED`, or `FAILED`.                                                                                              |
| `payload`     | JSON payload needed by the processor. Email payloads include the recipient and the template-specific details such as booking reference, secure link, or MFA code. |
| `attempts`    | Number of failed processing attempts.                                                                                                                             |
| `lastError`   | Last processing error message, if any.                                                                                                                            |
| `availableAt` | Earliest time this event should be retried.                                                                                                                       |
| `processedAt` | When the event was completed successfully.                                                                                                                        |
| `createdAt`   | When the event was created.                                                                                                                                       |
| `updatedAt`   | When the event was last changed.                                                                                                                                  |

Design decision:

The app uses the transactional outbox pattern for operational emails instead of sending email inline during the request. This prevents successful user actions, such as booking creation or password reset requests, from appearing to fail just because the email provider is unavailable.

## Session

The `Session` table stores refresh-token sessions.

Key fields:

| Field                 | Purpose                                                                                |
| --------------------- | -------------------------------------------------------------------------------------- |
| `id`                  | Primary key.                                                                           |
| `userId`              | User who owns the session.                                                             |
| `refreshToken`        | Unique stored refresh token value. In application code, this is intended to be hashed. |
| `familyId`            | Groups rotated refresh sessions that came from the same login.                         |
| `replacedBySessionId` | Replacement session created when this session was rotated.                             |
| `revokedAt`           | When the session stopped being active.                                                 |
| `revokedReason`       | Why the session was revoked, such as `ROTATED` or `REPLAY_DETECTED`.                   |
| `rememberMe`          | Whether the session came from a "keep me signed in" login.                             |
| `userAgent`           | Browser/device information.                                                            |
| `ipAddress`           | IP address associated with the session.                                                |
| `expiresAt`           | When the refresh session expires.                                                      |
| `createdAt`           | When the session was created.                                                          |
| `barberId`            | Optional link to a barber.                                                             |

Relationship:

```mermaid
erDiagram
    User ||--o{ Session : owns
    Barber ||--o{ Session : "optionally linked"

    Session {
        string id PK
        string userId FK
        string refreshToken UK
        string familyId
        string replacedBySessionId
        datetime revokedAt
        SessionRevocationReason revokedReason
        boolean rememberMe
        string userAgent
        string ipAddress
        datetime expiresAt
        datetime createdAt
        string barberId FK "optional"
    }
```

Design decision:

The application does not rely only on stateless JWTs. It also stores refresh-token sessions in the database. This supports logout, session invalidation, refresh-token rotation, and refresh-token replay detection.

Trade-off:

Database-backed sessions give more control and are safer for logout, but they add complexity. The API must correctly create, rotate, revoke, expire, and delete sessions.

Replay detection:

When refresh tokens rotate, the old session row is retained and marked as `ROTATED` instead of being deleted immediately. If that old refresh token is submitted again before it expires, the API treats it as replay and revokes active sessions in the same `familyId`.

Maintenance:

Expired sessions are deleted by a scheduled backend job. `Session.expiresAt` is indexed so the cleanup query remains efficient as the session table grows.

Current note:

The optional `barberId` on `Session` is unusual because a session already belongs to a `User`, and a barber is also linked to a user. If barber-specific sessions are needed, this may be useful. If not, it could be simplified later.

## ExternalAccount

The `ExternalAccount` table is designed for third-party authentication providers.

Key fields:

| Field        | Purpose                                             |
| ------------ | --------------------------------------------------- |
| `id`         | Primary key.                                        |
| `provider`   | Name of the external provider, such as Google.      |
| `providerId` | User identifier from that provider.                 |
| `userId`     | Local user account linked to the external provider. |
| `createdAt`  | When the external account link was created.         |

Relationship:

```mermaid
erDiagram
    User ||--o{ ExternalAccount : links

    ExternalAccount {
        string id PK
        string provider
        string providerId
        string userId FK
        datetime createdAt
    }
```

Design decision:

External accounts are separate from users, which allows one user to have multiple login providers in the future.

Trade-off:

This is flexible, but it adds a table that may not be needed until social login is actually implemented.

Important constraint:

`provider` and `providerId` are unique together. That prevents the same external account from being linked more than once.

## MFA

The current implemented MFA flow is email-code based.

`User` stores the active MFA preference:

| Field        | Purpose                                                              |
| ------------ | -------------------------------------------------------------------- |
| `mfaEnabled` | Whether the login flow should require MFA after password validation. |
| `mfaMethod`  | The configured method. Currently only `EMAIL` is supported.          |

`MfaChallenge` stores short-lived login challenges:

| Field        | Purpose                                                        |
| ------------ | -------------------------------------------------------------- |
| `id`         | Challenge id returned to the client after password validation. |
| `userId`     | The user this challenge belongs to.                            |
| `codeHash`   | Bcrypt hash of the 6-digit email code.                         |
| `method`     | MFA method used for the challenge.                             |
| `rememberMe` | Login persistence choice to apply after MFA succeeds.          |
| `expiresAt`  | When the challenge expires.                                    |
| `consumedAt` | Set when the challenge has been successfully used.             |
| `createdAt`  | When the challenge was created.                                |

The older `Mfa` table remains in the schema as a foundation for future TOTP-style MFA.

Key fields:

| Field       | Purpose                              |
| ----------- | ------------------------------------ |
| `id`        | Primary key.                         |
| `userId`    | The user this MFA record belongs to. |
| `secret`    | TOTP secret.                         |
| `enabled`   | Whether MFA is enabled.              |
| `createdAt` | When MFA was configured.             |

Relationship:

```mermaid
erDiagram
    User ||--o| Mfa : configures
    User ||--o{ MfaChallenge : receives

    Mfa {
        string id PK
        string userId UK, FK
        string secret
        boolean enabled
        datetime createdAt
    }

    MfaChallenge {
        string id PK
        string userId FK
        string codeHash
        MfaMethod method
        datetime expiresAt
        datetime consumedAt
        datetime createdAt
    }
```

Design decision:

The simple login decision lives on `User` through `mfaEnabled` and `mfaMethod`, while one-time challenge state lives in `MfaChallenge`.

Trade-off:

This keeps login checks cheap and keeps raw MFA codes out of the database, but it introduces a short-lived table that needs scheduled cleanup.

## Full Entity Relationship Diagram

```mermaid
erDiagram
    User {
        string id PK
        string name
        string email UK
        string passwordHash
        Role role
        datetime createdAt
        datetime updatedAt
        boolean isEmailVerified
        boolean mfaEnabled
        MfaMethod mfaMethod
    }

    Barber {
        string id PK
        string userId UK, FK
        string displayName UK
        boolean isActive
        string phone UK
        datetime createdAt
    }

    Booking {
        string id PK
        string userId FK
        string barberId FK "optional"
        datetime startTime
        datetime endTime
        datetime createdAt
    }

    Session {
        string id PK
        string userId FK
        string refreshToken UK
        string userAgent
        string ipAddress
        datetime expiresAt
        datetime createdAt
        string barberId FK "optional"
    }

    ExternalAccount {
        string id PK
        string provider
        string providerId
        string userId FK
        datetime createdAt
    }

    Mfa {
        string id PK
        string userId UK, FK
        string secret
        boolean enabled
        datetime createdAt
    }

    MfaChallenge {
        string id PK
        string userId FK
        string codeHash
        MfaMethod method
        datetime expiresAt
        datetime consumedAt
        datetime createdAt
    }

    User ||--o{ ExternalAccount : has
    User ||--o| Mfa : has
    User ||--o{ MfaChallenge : has
    User ||--o{ Session : owns
    User ||--o{ Booking : makes
    User ||--o| Barber : "can be"
    Barber ||--o{ Booking : receives
    Barber ||--o{ Session : "optionally linked"
```

## Relationship Summary

| Relationship              | Meaning                                                  |
| ------------------------- | -------------------------------------------------------- |
| `User -> ExternalAccount` | One user can have many external login providers.         |
| `User -> Mfa`             | One user can have one MFA setup.                         |
| `User -> MfaChallenge`    | One user can have many short-lived MFA login challenges. |
| `User -> Session`         | One user can have many active or historical sessions.    |
| `User -> Booking`         | One user can make many bookings.                         |
| `User -> Barber`          | One user can optionally have one barber profile.         |
| `Barber -> Booking`       | One barber can have many bookings.                       |
| `Barber -> Session`       | One barber can optionally be linked to many sessions.    |

## Constraints and Indexes

Important constraints:

- `User.email` is unique.
- `Barber.userId` is unique, so one user can only have one barber profile.
- `Barber.displayName` is unique.
- `Barber.phone` is unique.
- `Session.refreshToken` is unique.
- `Session.expiresAt` is indexed for expired-session cleanup.
- `Session.familyId` is indexed for session-family replay revocation.
- `ExternalAccount.provider + ExternalAccount.providerId` is unique as a pair.
- `MfaChallenge.userId` and `MfaChallenge.expiresAt` are indexed for challenge lookup and cleanup.
- `Booking.userId` has an index to make user booking lookups faster.
- `Booking.barberId + Booking.startTime` has a partial unique index for non-cancelled bookings to prevent double-booking the same barber slot.
- `OutboxEvent.status + OutboxEvent.availableAt` is indexed so the scheduled processor can find due work efficiently.
- `OutboxEvent.type + OutboxEvent.status` is indexed for event-type monitoring and operational queries.

Cascade behaviour:

- Deleting a `User` cascades to their `Barber` profile.
- Deleting a `User` cascades to their `Booking` records.

## Current Design Trade-Offs

The current schema is a good foundation for authentication and booking, but there are some intentional or likely trade-offs:

- A single `User` table with roles keeps authentication simple, but requires strong role checks in the API.
- `Barber` as a separate profile keeps barber-specific data out of `User`, but the app must keep `User.role` and `Barber` records aligned.
- Optional `Booking.barberId` keeps the schema flexible, but the current MVP booking flow should create bookings against a selected barber.
- Availability rules and exceptions keep normal working hours separate from one-off closures or extra availability.
- `Service` as a table supports prices, durations, descriptions, active/inactive service lifecycle, and historical booking reporting.
- Database-backed sessions support logout and refresh-token invalidation, but make auth more complex than purely stateless JWTs.
- `ExternalAccount` and `Mfa` suggest future-proofing, but they add schema surface before those flows are fully built.

## Mental Model

The easiest way to understand the database is:

1. Everyone starts as a `User`.
2. Some users may also become `Barber` records.
3. Customers create `Booking` records.
4. Bookings can optionally be assigned to barbers.
5. Authentication is supported by `Session`, `Mfa`, and `ExternalAccount`.
