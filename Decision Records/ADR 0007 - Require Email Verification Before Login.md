# ADR 0007 - Require Email Verification Before Login

Status: Accepted

Date: 2026-05-10

## Context

Users can register with an email and password. The application needs to know that the user actually owns the email address they used.

This matters because future booking flows may send booking confirmations, password reset links, reminders, or account notifications by email.

## Decision

Create users immediately after registration, but prevent login until their email address is verified.

The `User` table has:

`isEmailVerified`

The login use case rejects users whose email is not verified.

## Alternatives Considered

## Allow Login Before Verification

Pros:

- smoother signup experience
- user can start using the app immediately

Cons:

- fake or mistyped emails can create active accounts
- booking confirmations may go to the wrong address
- password reset and account recovery become less reliable

## Do Not Create User Until Verification

Pros:

- database only stores verified users
- no cleanup needed for abandoned registrations

Cons:

- registration flow becomes more complex
- pending registration state has to be stored somewhere
- harder to resend verification emails

## Use Magic Links Only

Pros:

- no password storage
- email ownership is naturally verified by login

Cons:

- user experience depends heavily on email delivery
- not everyone likes passwordless login
- less aligned with the current password-based auth implementation

## Decision Rationale

Creating the user first and requiring verification is a common and practical approach.

It allows:

- password-based registration
- resend verification email flow
- clear user state in the database
- future login and booking flows to rely on verified email addresses

## Trade-Offs

The main trade-off is that unverified users can remain in the database.

That is acceptable during early development, but a production system may need a cleanup policy for users who never verify.

## Consequences

Positive consequences:

- safer account onboarding
- reliable email identity before login
- clear user state through `isEmailVerified`

Negative consequences:

- extra step in the user journey
- email delivery becomes part of the critical signup path
- abandoned unverified accounts may accumulate

## Current Follow-Up Work

- improve post-verification frontend auth behavior
- add cleanup policy for stale unverified accounts if needed
- add clear UI messaging when login fails due to unverified email
- test resend verification behavior

