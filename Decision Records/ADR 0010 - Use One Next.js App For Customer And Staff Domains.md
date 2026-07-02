# ADR 0010 - Use One Next.js App For Customer And Staff Domains

Status: Accepted

Date: 2026-07-01

## Context

The application needs separate customer and staff experiences.

Customer-facing pages include the public site, registration, login, services, bookings, and account management.

Staff-facing pages include barber login, dashboard, calendar, bookings, availability, customers, and settings.

The project also uses subdomains:

- the root/customer domain for customers
- a staff subdomain, such as `staff.ericsbarbers.com`, for barbers and admins
- an optional test staff subdomain, such as `test-staff.ericsbarbers.com`, for staging or preview-style testing

The frontend is already a Next.js application deployed through Vercel, and Vercel can route multiple configured domains to the same project.

## Decision

Use one Next.js project for both customer and staff surfaces.

Organize routes by internal app folders:

```text
app/customer
app/staff
```

Use `proxy.ts` as the host-aware routing boundary:

- customer-domain requests rewrite to internal `/customer/*` routes
- staff-domain requests rewrite to internal `/staff/*` routes
- the browser keeps clean public URLs such as `/login`, `/dashboard`, and `/calendar`

Examples:

```text
ericsbarbers.com/login              -> /customer/login
ericsbarbers.com/services           -> /customer/services
staff.ericsbarbers.com/login        -> /staff/login
staff.ericsbarbers.com/dashboard    -> /staff/dashboard
test-staff.ericsbarbers.com/login   -> /staff/login
```

Staff hosts are identified from:

```text
NEXT_PUBLIC_STAFF_SITE_URL
NEXT_PUBLIC_TEST_STAFF_SITE_URL
```

The proxy also supports local staff hostnames such as:

```text
staff.localhost
test-staff.localhost
```

## Alternatives Considered

## Separate Next.js Apps

Pros:

- hard separation between customer and staff UI code
- separate deployments and environment variables
- fewer host-aware routing rules

Cons:

- more projects to maintain
- duplicated auth UI and shared components
- more deployment configuration for a solo project
- harder to keep customer and staff design primitives aligned

## One App With Only Path-Based Routes

Example:

```text
ericsbarbers.com/staff/login
```

Pros:

- simpler DNS and local development
- no host-aware routing needed
- one domain for all user types

Cons:

- less clear product separation
- staff workflows feel like a subsection of the public customer site
- less suitable if staff links should live on a dedicated subdomain

## Wildcard Or Per-Barber Subdomains

Example:

```text
tom.ericsbarbers.com
```

Pros:

- could support public barber profiles or multi-tenant-style routing later

Cons:

- unnecessary complexity for the current product
- wildcard DNS and certificate handling are more involved
- would require tenant lookup and stronger routing rules

## Decision Rationale

One Next.js app keeps the project maintainable while still supporting a clean staff subdomain.

This is a good fit because:

- customer and staff surfaces share auth infrastructure
- the app can reuse common UI components
- Vercel can point multiple domains at one project
- Next.js proxy logic can rewrite by host before rendering
- the staff subdomain improves navigation without requiring a second frontend deployment

## Trade-Offs

The frontend now has a host-aware routing layer.

The team must understand that:

- public paths and internal App Router paths are not always the same
- local development may require hosts-file entries such as `staff.localhost`
- staff subdomain routing is a UX boundary, not an authorization boundary
- backend role checks must still protect staff and admin APIs

## Consequences

Positive consequences:

- the customer and staff experiences can evolve in one frontend project
- clean staff URLs can be used without duplicating the app
- staging staff URLs such as `test-staff.*` can use the same internal staff routes
- shared auth route handlers and components stay centralized

Negative consequences:

- `proxy.ts` is more important and needs focused tests
- domain-specific bugs can appear if host headers are not handled correctly
- local development setup needs documentation for `staff.localhost`

## Follow-Up Work

- document local staff subdomain setup in the local development guide
- keep proxy tests covering customer, staff, production, and local hostnames
- consider whether staff navigation should be hidden from customer-domain rendering
- add backend role guards for staff and admin API endpoints as those workflows become real
