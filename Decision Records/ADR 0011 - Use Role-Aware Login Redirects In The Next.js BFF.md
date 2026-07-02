# ADR 0011 - Use Role-Aware Login Redirects In The Next.js BFF

Status: Accepted

Date: 2026-07-01

## Context

The application now has customer and staff login surfaces.

The NestJS backend issues JWT access tokens that include the user's role in the access-token payload.

The frontend BFF route handlers already receive the access token after login and MFA verification so they can set HttpOnly cookies on the frontend domain.

Without role-aware redirects, successful login would always send users to the same page, such as `/my-account`, regardless of whether the account is a customer, barber, or admin.

## Decision

Decode the access token in the Next.js BFF after successful login and MFA verification.

Use the decoded role to return a `redirectTo` value to the browser.

Current redirect intent:

| Role | Customer Domain Login | Staff Domain Login |
| --- | --- | --- |
| `CUSTOMER` | `/my-account` | customer domain `/my-account` |
| `BARBER` | staff domain `/dashboard` | `/dashboard` |
| `ADMIN` | staff domain `/dashboard` | `/dashboard` |

The BFF response shape is:

```json
{
  "message": "Logged in",
  "redirectTo": "/my-account",
  "role": "CUSTOMER"
}
```

The browser uses `redirectTo` for navigation after login or MFA verification.

## Alternatives Considered

## Always Redirect In The Client

The client page could call `/api/auth/profile` after login, read the role, and choose a destination.

Pros:

- avoids decoding the token in the BFF
- uses a profile endpoint as the source of user data

Cons:

- adds an extra request during login
- makes login navigation depend on an additional API call
- duplicates role redirect rules in client-side code

## Backend Returns Redirect Target Directly

The NestJS API could return a role-specific redirect target.

Pros:

- backend owns more of the post-login decision
- role data is already available during token creation

Cons:

- backend would need to know frontend domain configuration
- frontend deployment/domain concerns would leak into the API
- harder to support customer and staff domains cleanly from the backend alone

## Proxy-Only Redirects

The proxy can already redirect authenticated users away from login routes based on host.

Pros:

- centralizes some navigation rules in one frontend boundary

Cons:

- proxy does not verify token signatures
- proxy only sees cookies on navigation, not the login response body
- proxy cannot replace the post-login client navigation decision

## Decision Rationale

The Next.js BFF is the right boundary for this decision because:

- it already receives the access token
- it already owns browser-facing cookie setting
- it knows the request host
- it can use frontend domain configuration
- it can keep backend auth logic separate from frontend navigation concerns

Decoding the access token in the BFF is used only for navigation. It is not treated as the final authorization boundary.

## Trade-Offs

The BFF now depends on the access token containing a stable `role` claim.

If token claims change, the BFF redirect logic must be updated.

The BFF decodes the token payload for navigation, but protected backend operations must still verify the token and enforce roles server-side.

## Consequences

Positive consequences:

- customers and staff can share the same login API route
- staff users are sent to the staff dashboard after login
- customers who log in from the staff domain can be sent back to the customer account area
- the login UI no longer hard-codes one destination for every role

Negative consequences:

- the BFF has more auth-adjacent logic
- tests must cover role and host combinations
- role names in token claims become part of the frontend contract

## Follow-Up Work

- include the role contract in API/auth documentation
- add role-aware behavior to future password reset or invite flows if needed
- keep backend role guards as the real security boundary
- consider dedicated admin landing behavior if admin workflows diverge from barber workflows
