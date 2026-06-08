# ADR 0009 - Use Feature Flags For Booking Availability

Status: Accepted

Date: 2026-05-10

## Context

The booking feature is not complete yet, but the project already has some booking routes, pages, and backend structure.

It is useful to keep unfinished work in the codebase while preventing users from relying on incomplete functionality.

## Decision

Use feature flags to control booking availability.

Frontend flag:

`NEXT_PUBLIC_BOOKING_ENABLED`

Backend flag:

`BOOKING_ENABLED`

## Alternatives Considered

## Leave Booking Routes Visible While Incomplete

Pros:

- easier to test during development
- less conditional logic

Cons:

- users may hit broken flows
- incomplete features look like bugs
- harder to launch the auth portion safely

## Remove Booking Code Until Ready

Pros:

- no unfinished feature surface
- simpler current app

Cons:

- harder to iterate on booking gradually
- loses useful scaffolding
- repeated work when reintroducing booking routes

## Use Branches Only For Incomplete Features

Pros:

- main branch only contains finished work
- cleaner production code

Cons:

- long-running branches drift
- harder to integrate incrementally
- less useful if frontend placeholders and backend scaffolding are already part of the current app

## Decision Rationale

Feature flags allow the project to keep booking foundations in place while hiding or blocking incomplete user-facing behavior.

The frontend can show a "coming soon" state when booking is disabled.

The backend can reject booking API requests when booking is disabled.

## Trade-Offs

Feature flags add configuration complexity. Both frontend and backend flags need to be set consistently.

There is also a risk that the frontend says booking is enabled while the backend rejects booking requests, or the backend is enabled while the frontend hides the feature.

## Consequences

Positive consequences:

- incomplete booking work can remain in the codebase
- safer partial releases
- easy to turn booking on when ready
- clearer user-facing messaging

Negative consequences:

- duplicated flags across frontend and backend
- more environment variables to manage
- developers need to remember to test both enabled and disabled states

## Current Follow-Up Work

- decide when booking is ready to enable
- define the minimum booking feature set
- test frontend and backend behavior with flags on and off
- consider centralizing feature flag configuration if the app grows

