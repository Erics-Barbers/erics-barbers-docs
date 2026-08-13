# ADR 0025 - Establish The NestJS OpenAPI Document As The Canonical Client Contract

Status: Accepted

Date: 2026-08-13

## Context

The Next.js repository contained a manually copied OpenAPI document, while the NestJS repository exposed runtime Swagger UI without a deterministic export command. The copied document omitted implemented guest-booking routes and most response schemas, and there was no reliable way to detect drift before generating web or mobile clients.

Mobile development requires one trustworthy contract for both clients. Maintaining separate web and mobile descriptions would allow the same endpoint to acquire incompatible types and behaviour.

## Decision

The NestJS repository owns `openapi/openapi.json` as the canonical committed client contract.

Runtime Swagger UI and the committed document shall use the same document factory. The repository shall provide deterministic generate and check commands. Client repositories shall consume a copied or published form of this canonical artifact and shall not independently edit their copy.

The web repository retains its current copied `api/api-spec.json` path for compatibility with its existing generator, but that file is synchronized from the API artifact. The mobile client will consume the same API-owned document when generated client integration begins.

## Alternatives Considered

### Keep The Web Copy As The Source

This preserves the existing generator path but puts contract ownership outside the server implementation and requires manual reconciliation after every controller change.

### Fetch Swagger JSON From A Running Environment

This avoids committing an artifact but makes builds depend on network/environment availability and can silently generate clients against a deployment that differs from the checked-out API code.

### Maintain Separate Web And Mobile Specifications

This permits client-specific views but duplicates shared semantics and creates precisely the contract drift the architecture is intended to prevent.

## Rationale

Generating the artifact from NestJS controllers and DTO metadata keeps contract ownership next to runtime behaviour. A committed, deterministically sorted artifact supports review, offline generation, CI drift checks, and stable client builds.

## Trade-Offs

Controllers and response DTOs require deliberate Swagger annotations. Client copies must be synchronized when the canonical file changes until artifact publication is automated.

## Consequences

- API changes that affect clients must update and verify the canonical document.
- Runtime Swagger and generated clients share the same description.
- Web and mobile clients have a common contract source.
- Drift checks can fail before stale generated code is merged.

## Follow-Up Work

- add `npm run openapi:check` to API continuous integration;
- automate synchronization or publication of the canonical artifact;
- generate the mobile API client from this artifact after the native authentication contract is accepted.
