# ADR 0022 - Build A Customer Mobile App With React Native And Expo

Status: Accepted

Date: 2026-08-10

## Context

Eric's Barbers currently serves browser users through a host-aware Next.js application. Browser authentication uses Next.js route handlers as a backend-for-frontend (BFF), with access and refresh tokens stored in HttpOnly cookies on the frontend domain.

The project also needs a native mobile experience. A native client has different navigation, secure-storage, linking, application-lifecycle, and authentication-transport requirements from the browser client. Reusing the Next.js BFF or browser UI would preserve assumptions that do not fit a native application.

The booking API already supports public service and barber discovery, availability lookup, authenticated bookings, and guest booking management by reference. The API contract and committed OpenAPI document have some drift that must be resolved before generated mobile client code becomes a reliable foundation.

## Decision

Build a React Native mobile application using Expo and Continuous Native Generation (CNG) in the `erics-barbers-app` repository.

Native iOS and Android projects will initially be generated when needed rather than maintained as permanent source files. Native configuration will be expressed through Expo app configuration and config plugins. The generated `ios` and `android` directories will remain outside source control unless a later accepted architectural decision establishes a need to maintain them directly.

Version 1 will be a customer application only. Staff and admin mobile experiences are outside the initial scope.

This does not allocate staff or administration interfaces to a later mobile version. Any such expansion requires a separate business case, requirements baseline, UX and security analysis, and an accepted decision about whether it belongs in the customer application, a separate application, or the responsive staff website.

The mobile application will:

- communicate directly with the NestJS API rather than through the Next.js BFF
- use bearer access tokens and an intentional native refresh-token contract
- store sensitive durable credentials with Expo SecureStore
- keep the access token and active session state in memory where practical
- use TanStack Query for server state
- use React and form state for transient UI and booking-flow state
- support both authenticated and guest booking journeys
- preserve booking management by high-entropy booking reference
- use universal/app links for email verification, password reset, and other relevant customer links, with an appropriate web fallback

The existing Next.js application will retain its BFF and HttpOnly-cookie authentication model.

API/OpenAPI contract drift must be addressed before mobile features depend on generated API code.

Push notifications are deferred to a later mobile version and will be designed as a dedicated backend and mobile capability. Payments are also deferred until the core booking lifecycle is stable.

## Alternatives Considered

## Reuse The Next.js BFF From Mobile

Pros:

- preserves the existing browser authentication boundary
- requires fewer immediate changes to the NestJS authentication endpoints

Cons:

- couples the native app to browser-oriented cookie and routing behavior
- makes secure native token storage and application lifecycle restoration indirect
- adds an unnecessary network and deployment dependency
- obscures which service owns the native API contract

## Wrap The Existing Website In A WebView

Pros:

- fastest way to display the existing customer UI on a phone
- maximizes reuse of web components

Cons:

- does not provide a first-class native experience
- complicates native linking, secure storage, navigation, and future push notifications
- carries existing browser layout and state-management constraints into the app

## Include Customer And Staff Features In Version 1

Pros:

- one mobile release could serve both audiences
- creates a broader initial feature set

Cons:

- the staff dashboard currently relies on sample data
- barber booking and availability-management APIs are incomplete
- substantially increases authentication, authorization, navigation, and testing scope
- delays a coherent customer release

## Decision Rationale

React Native and Expo provide a suitable cross-platform foundation for iOS and Android while allowing native secure storage, linking, navigation, and future notification support.

A customer-only first version matches the most mature product and API surface. Direct NestJS communication gives mobile a deliberate contract without weakening or replacing the browser BFF. Retaining guest booking preserves the low-friction booking model already accepted for the web product.

## Trade-Offs

- Some user journeys and validation behavior can be reused conceptually, but Next.js components cannot be reused directly.
- Native changes need to be represented through Expo configuration, config plugins, or native modules so generated projects remain reproducible.
- The backend must support a native refresh and logout transport in addition to the existing cookie transport.
- Universal links require coordinated mobile, backend, email-template, and website configuration.
- The project will maintain separate web and mobile presentation layers.
- Contract generation becomes more important because multiple clients depend on the API.

## Consequences

Positive consequences:

- the mobile app has a native architecture and lifecycle
- customer mobile work can proceed independently of unfinished staff tooling
- the Next.js BFF remains optimized for browser security
- guest and authenticated customers retain equivalent booking options
- future push notifications have a suitable native foundation

Negative consequences:

- authentication now needs two explicit transport adapters
- API response schemas and OpenAPI generation require additional maintenance
- universal-link behavior must be tested across installed-app and web-fallback scenarios
- business behavior must remain consistent across two customer interfaces

## Follow-Up Work

- decide and document the native login, refresh, logout, and token-revocation API contract
- resolve current API/OpenAPI contract drift and automate drift detection
- define universal-link domains, routes, fallback behavior, and token-handling rules
- define the version 1 customer screen and navigation map
- maintain the release-neutral [[Mobile App Requirements]] and [[Mobile App Delivery Roadmap]] documents
- decide whether native staff or administration interfaces are needed before assigning requirements or a target version
- scaffold the Expo application only after the foundational contracts are stable
- plan push-token registration, notification preferences, and reminder delivery in a later ADR
