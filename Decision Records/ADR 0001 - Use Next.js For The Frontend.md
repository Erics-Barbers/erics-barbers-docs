# ADR 0001 - Use Next.js For The Frontend

Status: Accepted

Date: 2026-05-10

## Context

Eric's Barbers needed a frontend that could support a real web application rather than just a static portfolio-style site. The application needed pages for registration, login, email verification, account management, services, and eventually bookings.

The project also needed room to grow into server-side behavior, especially around authentication. Handling cookies securely was an important part of the authentication flow.

## Decision

Use Next.js as the main frontend framework.

The active frontend project is:

`erics-barbers-ui`

## Alternatives Considered

## Plain React With Vite

This would have been simpler and faster to start with. There is already an older `erics-barbers-ui-react` project that appears to be a Vite React starter.

Pros:

- simpler mental model
- fast development server
- fewer framework concepts
- good fit for a purely client-rendered app

Cons:

- no built-in server-side route handlers
- auth cookie handling would need a different backend-for-frontend approach
- routing and protected pages would need more manual setup
- less practice with full-stack React patterns

## Traditional Server-Rendered App

The app could have been built with server-rendered templates from the backend.

Pros:

- simpler deployment shape
- backend owns most auth and rendering concerns
- fewer moving pieces

Cons:

- less interactive frontend experience
- less aligned with modern React/full-stack development practice
- harder to build rich future booking interfaces

## Decision Rationale

Next.js gave the project a good balance between frontend React development and server-side capabilities.

The key reasons were:

- file-based routing through the app directory
- React component model
- server-side route handlers
- ability to manage HttpOnly cookies on the frontend domain
- good fit for a production-style full-stack project
- useful learning value for modern full-stack development

## Trade-Offs

The main trade-off is complexity. Next.js is not just a frontend library. It introduces server/client boundaries, route handlers, server rendering, middleware/proxy behavior, and cookie handling rules.

That complexity showed up most clearly in authentication, where it became necessary to think carefully about where cookies live and which layer is responsible for setting or forwarding them.

## Consequences

Positive consequences:

- the app can use React for UI and Next.js route handlers for server-side auth helpers
- protected frontend routes can be handled before rendering
- the project better resembles a real production web app

Negative consequences:

- developers must understand whether code runs in the browser or on the Next.js server
- auth flow is more complex than a purely client-rendered app
- the frontend now has some backend-like responsibilities

## Current Follow-Up Work

- consolidate when to use generated API client calls versus Next.js API routes
- improve auth cookie handling
- document server/client boundaries in frontend code as the app grows

## Status Update - 2026-07-01

The Next.js frontend now serves both customer and staff surfaces from one project.

Customer-facing pages live under `app/customer`, and staff-facing pages live under `app/staff`.

The frontend uses host-aware proxy routing so the root customer domain maps to customer routes and the staff subdomain maps to staff routes.

Related decision:

- [[ADR 0010 - Use One Next.js App For Customer And Staff Domains]]
