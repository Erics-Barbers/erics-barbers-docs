# ADR 0006 - Use Next.js API Routes For Frontend Auth Cookies

Status: Accepted, needs refinement

Date: 2026-05-10

## Context

The frontend needed to store the access token safely after login.

Storing the access token in `localStorage` would be simple, but it would expose the token to client-side JavaScript. A safer approach is to use an HttpOnly cookie that client-side JavaScript cannot read.

Because the frontend is built with Next.js, route handlers can set cookies on the frontend domain.

## Decision

Use Next.js API routes as a frontend authentication boundary for selected auth flows.

Current examples:

- `app/api/auth/login/route.ts`
- `app/api/auth/profile/route.ts`
- `app/api/auth/logout/route.ts`

## Alternatives Considered

## Store Access Token In LocalStorage

Pros:

- simple to implement
- easy to add Authorization headers from client-side code
- common in small demos

Cons:

- exposed to JavaScript
- more vulnerable if an XSS issue exists
- not ideal for a production-style app

## Store Access Token In React State Only

Pros:

- avoids persistent token storage
- token disappears on refresh

Cons:

- poor user experience
- hard to support page reloads
- still needs refresh logic

## Let Backend Set All Cookies Directly

Pros:

- backend fully owns auth cookies
- fewer frontend auth routes

Cons:

- cross-site cookie behavior becomes more complicated
- frontend and backend may be on different domains
- the Next.js app still needs a convenient way to protect pages

## Decision Rationale

Using Next.js API routes lets the frontend server receive the access token from the backend and set it as an HttpOnly cookie for the frontend domain.

This supports:

- protected frontend routes through `proxy.ts`
- less token exposure to browser JavaScript
- a clearer frontend boundary for login/profile/logout behavior

## Trade-Offs

The trade-off is that the frontend now has server-side auth responsibilities.

The system has to manage:

- backend refresh-token cookie
- frontend access-token cookie
- cookie paths
- secure and same-site settings
- forwarding cookies between layers

This is more complicated than direct browser-to-backend calls.

## Consequences

Positive consequences:

- access token can be stored as HttpOnly
- protected frontend pages can be checked before rendering
- frontend auth behavior is easier to centralize in route handlers

Negative consequences at the time of decision:

- not all auth flows used the same route-handler pattern
- logout needs careful cookie forwarding
- refresh-token flow was not fully wired yet
- local development can be awkward with `secure: true` cookies

## Current Follow-Up Work

- decide whether all auth flows should go through Next.js route handlers
- add a local refresh route if refresh-token sessions remain part of the design
- make logout clear both frontend and backend auth state reliably
- document cookie behavior for local and production environments

## Status Update - 2026-06-12

Implemented since this ADR:

- browser-facing auth flows now go through Next.js BFF route handlers
- registration, login, resend verification, email verification, profile, and logout use local `/api/auth/*` routes
- the BFF sets both `accessToken` and `refreshToken` cookies on the UI domain
- logout always clears local auth cookies, even if the backend logout call fails
- the account page redirects to the homepage after a logout click
- the NestJS logout endpoint is idempotent for missing, malformed, wrong-type, expired, or already-revoked refresh tokens
- the proxy protects current and anticipated private route prefixes
- focused Jest tests cover proxy and auth route-handler behavior

Remaining follow-up work:

- build the password reset UI/BFF flow
- keep generated auth client usage out of browser auth flows

## Status Update - 2026-07-01

Implemented since the previous update:

- the login and MFA BFF routes now decode the returned access token to read the user's role
- successful login responses include a `redirectTo` value for role-aware navigation
- customer and staff login pages share the same login flow component
- customer and staff domains keep host-scoped frontend auth cookies by default
- staff login on the staff subdomain redirects barbers and admins to the clean staff dashboard path

Related decision:

- [[ADR 0011 - Use Role-Aware Login Redirects In The Next.js BFF]]
