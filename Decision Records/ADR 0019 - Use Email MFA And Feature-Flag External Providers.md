# ADR 0019 - Use Email MFA And Feature-Flag External Providers

Status: Accepted

Date: 2026-07-02

## Context

The authentication module needs a modest MFA option without overbuilding social login or provider integrations too early.

The app is small-scale, and the immediate value is stronger account protection for users who opt in. External providers such as Google may be useful later, but they are not necessary for the current customer and staff login flow.

The data model already has room for MFA and external account identity records.

## Decision

Implement MFA as an email-code challenge after successful password validation.

Current behavior:

- `User.mfaEnabled` controls whether login requires MFA
- `User.mfaMethod` stores the selected method, currently `EMAIL`
- login returns `MFA_REQUIRED` instead of access and refresh tokens when MFA is enabled
- the API creates a short-lived `MfaChallenge`
- the challenge stores a hashed MFA code and the selected `rememberMe` value
- only successful MFA verification issues access and refresh tokens
- expired MFA challenges are removed by scheduled cleanup

Keep external providers behind configuration:

```text
AUTH_EXTERNAL_PROVIDERS_ENABLED=false
```

Do not expose provider buttons in the UI unless the feature flag is enabled.

Do not build full Google login until there is a clear product need.

Keep the `ExternalAccount` model as a foundation for linking provider identities later.

## Alternatives Considered

## Build Google Login Immediately

Pros:

- familiar login option for many users
- useful learning exercise
- reduces password-only account creation

Cons:

- adds provider setup, callbacks, account linking, and edge cases
- increases auth surface area before the core product is stable
- not clearly needed for a small booking app yet

## Build TOTP Authenticator MFA First

Pros:

- stronger than email-code MFA
- common for admin/staff accounts

Cons:

- more UI and recovery-flow complexity
- users need authenticator apps
- overbuilt for the current customer login experience

## No MFA Until Later

Pros:

- simpler login flow
- fewer tables and cleanup jobs

Cons:

- delays a useful security feature
- staff/admin workflows will eventually need stronger login controls
- harder to retrofit cleanly once auth flows spread

## Decision Rationale

Email MFA is the smallest useful MFA implementation for this project.

It fits the current architecture because:

- operational emails already go through the backend email path
- the API owns credential validation and token issuing
- the BFF can keep browser cookie behavior unchanged
- `rememberMe` can be preserved through the MFA challenge

Feature-flagging external providers keeps the data model open without committing to provider behavior before it is needed.

## Trade-Offs

Email MFA is weaker than authenticator-app MFA because email account compromise can bypass it.

It also depends on reliable email delivery.

However, it is a pragmatic starting point and can later be complemented with TOTP or provider-based login.

## Consequences

Positive consequences:

- MFA-enabled users do not receive tokens until the second factor succeeds
- the login flow supports `rememberMe` both with and without MFA
- the system has a clear place to add future MFA methods
- external login remains possible later without appearing prematurely in the UI

Negative consequences:

- login has an extra challenge state
- expired challenge cleanup must keep running
- tests need to cover login, MFA verification, expiry, and resend/attempt behavior as it grows

## Follow-Up Work

- consider staff/admin defaults for MFA once those workflows are live
- decide whether TOTP should replace or supplement email MFA later
- add recovery and backup-code policy before requiring MFA for high-privilege users
- implement provider login only when product usage justifies it
