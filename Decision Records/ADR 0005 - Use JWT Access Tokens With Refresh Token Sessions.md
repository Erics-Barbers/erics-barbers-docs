# ADR 0005 - Use JWT Access Tokens With Refresh Token Sessions

Status: Accepted, needs refinement

Date: 2026-05-10

## Context

The application needs users to stay logged in without sending their password on every request.

It also needs a way to protect backend endpoints, identify the current user, and support logout.

## Decision

Use short-lived JWT access tokens with longer-lived refresh tokens stored as database-backed sessions.

Access tokens:

- last 15 minutes
- are used as Bearer tokens for protected API calls
- include user identity through `sub`

Refresh tokens:

- last 7 days
- are intended to be stored as HttpOnly cookies
- are recorded in the `Session` table
- can be invalidated on logout

## Alternatives Considered

## Stateless JWT Only

Pros:

- simpler backend
- no session table needed
- easy horizontal scaling

Cons:

- logout is difficult
- token revocation is difficult
- stolen tokens remain valid until expiry

## Server-Side Sessions Only

Pros:

- simpler revocation
- familiar web authentication model
- no need to manage JWT payloads

Cons:

- needs session storage for every logged-in request
- less natural for APIs consumed by multiple clients
- requires careful cookie/session configuration

## Third-Party Auth Provider

Examples: Auth0, Clerk, Supabase Auth, Firebase Auth.

Pros:

- faster to get production-grade auth
- less security-sensitive custom code
- built-in flows for password reset, MFA, social login

Cons:

- less learning value
- vendor dependency
- pricing and configuration complexity
- less control over the authentication model

## Decision Rationale

JWT access tokens plus refresh sessions gave the project a realistic authentication model while still allowing the backend to own user identity.

This approach supports:

- short-lived access tokens
- longer-lived sessions
- database-backed logout
- future multi-device session management

It also gave useful learning value because it required thinking about tokens, cookies, sessions, and trust boundaries.

## Trade-Offs

This design is more complex than a simple session cookie or a stateless JWT.

The backend must:

- issue tokens
- verify tokens
- store refresh-token sessions
- rotate refresh tokens
- invalidate sessions on logout
- handle expired sessions

The frontend must:

- store or receive tokens safely
- send access tokens to protected endpoints
- refresh sessions when access tokens expire
- clear auth state on logout

## Consequences

Positive consequences:

- access tokens are short-lived
- refresh sessions can be invalidated
- logout can remove server-side session state
- the model can grow toward multi-device session management

Negative consequences:

- refresh-token handling is easy to get wrong
- the current implementation needs review around bcrypt-hashed refresh token lookup
- frontend refresh flow is not complete yet
- cookie forwarding between Next.js and the backend needs careful handling

## Current Follow-Up Work

- review refresh-token storage and lookup strategy
- complete the frontend refresh-token flow
- make logout reliably invalidate backend sessions
- add tests for token rotation, reuse, expiry, and logout

## Status Update - 2026-06-12

Implemented since this ADR:

- refresh tokens are hashed in session rows and compared with bcrypt
- refresh tokens are rotated and both new tokens are returned to the Next.js BFF
- the Next.js proxy and profile BFF route can refresh expired access tokens
- email verification and password reset now use dedicated JWT `tokenType` values
- focused frontend tests cover proxy refresh behavior and auth route-handler cookie behavior
- API logout is idempotent for missing, malformed, wrong-type, expired, or already-revoked refresh tokens
- refresh-token rotation deletes the old session and creates the new session in one database transaction

Remaining follow-up work:

- decide whether to add session-family replay detection
- add expired session cleanup
