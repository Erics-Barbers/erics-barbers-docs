# Delivery Workflow

Status: accepted working agreement

Version: 1.2

Recorded: 12 August 2026

## Purpose

This document defines the lightweight ticket workflow used to preserve development context across sessions. The workflow is designed for a side project in which the user performs most implementation while the assistant maintains the operational record during normal conversations.

Requirements, roadmaps, current-state documents, and ADRs remain the durable product and architecture sources. GitHub Issues and Projects answer the operational questions: what is ready, what is being worked on, what is blocked, and what should happen next.

## GitHub Structure

- Organization: `Erics-Barbers`
- Mobile Project: private [Erics Barbers - Mobile App Delivery](https://github.com/orgs/Erics-Barbers/projects/1)
- Web Project: private [Erics Barbers - Web App Delivery](https://github.com/orgs/Erics-Barbers/projects/2)
- Issues live in the repository that owns the change.
- Cross-repository delivery is aggregated in the relevant organization Project and umbrella issue.
- The Mobile 1.0 baseline is catalogued in [[Mobile App Delivery Backlog]].
- The web release baseline is catalogued in [[Web App Delivery Backlog]] and aggregated in the [web delivery umbrella issue](https://github.com/Erics-Barbers/erics-barbers-docs/issues/6).

Repository ownership follows these boundaries:

| Repository | Ticket ownership |
| --- | --- |
| `erics-barbers-app` | React Native application, native configuration, mobile tests, and mobile release artifacts. |
| `erics-barber-api` | NestJS contracts, authentication transport, business rules, persistence, and backend tests. |
| `erics-barbers-ui` | Next.js web fallback pages, hosted association files, and browser-client compatibility. |
| `erics-barbers-docs` | Product decisions, requirements, roadmaps, ADRs, release procedures, and cross-repository coordination. |

## Project Fields

The Project should expose:

| Field | Values or purpose |
| --- | --- |
| Status | `Backlog`, `Ready`, `In progress`, `Blocked`, `Review / verify`, `Done` |
| Increment / Release | Mobile uses `[A]`-`[F]`; web uses `Web 0.x`-`Web 3.0`; stable catalogue IDs preserve identity across repositories. |
| Priority | GitHub's `Urgent`, `High`, `Medium`, and `Low`; catalogue priorities map as `P0` → `Urgent`, `P1` → `High`, and `P2` → `Medium`. |
| Effort | GitHub's `Low`, `Medium`, and `High`; catalogue sizes map as `S` → `Low`, `M` → `Medium`, and `L` → `High`. |

Useful views are:

- **Current** — board grouped by Status, excluding Done;
- **Roadmap** — table grouped by Increment and sorted by Priority;
- **Blocked** — tickets whose Status is Blocked; and
- **Completed** — tickets whose Status is Done.

GitHub Free permits only one built-in auto-add workflow. Routine assistant-managed issue creation should add the issue directly to the Project instead of depending on repository-specific auto-add workflows.

## Ticket Shape

Every delivery ticket should use this structure:

```markdown
## Outcome

The independently valuable or risk-reducing result.

## Requirement coverage

Requirement identifiers and source-document links.

## Scope

- Work included by this ticket.

## Acceptance criteria

- [ ] Observable evidence required before completion.

## Dependencies

Related tickets, accepted decisions, or external prerequisites.

## Handover

**Current state:** Not started.

**Next action:** The first concrete action.

**Blockers:** None.
```

Ticket titles begin with the roadmap increment in square brackets, followed by a concise lower-case action when applicable, for example `[C] implement guest booking confirmation`.

## Assistant-Managed Operation

During ordinary collaboration, the assistant should:

1. determine whether the conversation advances an existing ticket or creates a new independently trackable outcome;
2. create or select the owning-repository issue without requiring the user to maintain the board;
3. move the selected issue to `In progress` when implementation actually begins;
4. add concise decisions, evidence, changed scope, and blockers as work develops;
5. leave `Current state`, `Next action`, and `Blockers` usable whenever work pauses or the conversation changes focus;
6. move completed implementation to `Review / verify` until its acceptance evidence has passed; and
7. close the issue and mark it `Done` only when the result is genuinely complete.

The assistant may perform routine ticket writes under the user's standing delegation. It must involve the user before a ticket change would materially expand, remove, defer, or reprioritize accepted product scope.

## Work-in-Progress Rule

Keep one implementation ticket `In progress` at a time. Documentation or review work may overlap only when it directly supports that ticket or when the user explicitly chooses parallel work.

When focus changes, update the previous ticket before switching. This is the key mechanism that makes returning after several days reliable.

## Completion Evidence

Completion evidence is proportional to the ticket and can include:

- automated tests or contract checks;
- successful iOS and Android verification;
- screenshots or recorded manual scenarios;
- API response and error-contract verification;
- accessibility checks;
- updated documentation or ADRs; and
- linked pull requests and commits.

Writing code alone is not completion when the ticket requires integration or platform verification.

## Information Safety

Issues and Project fields must not contain credentials, access or refresh tokens, verification or reset tokens, secure booking references, customer personal information, or sensitive production logs. Use redacted and synthetic examples when diagnostic context is necessary.

## Change Control

The ticket catalogue may be refined as implementation reveals better boundaries. Splitting or combining tickets does not change requirements by itself. Any change to accepted product behaviour or release allocation must also update the appropriate requirements or roadmap, and significant architectural decisions must follow the ADR process.

## Version History

| Version | Date | Change |
| --- | --- | --- |
| 1.2 | 18 September 2026 | Added the live web delivery Project and generalized fields for mobile increments and web releases. |
| 1.1 | 12 August 2026 | Added the live GitHub Project and aligned field guidance with the Project's available Priority and Effort fields. |
| 1.0 | 12 August 2026 | Established the assistant-managed GitHub Issues and Projects workflow. |
