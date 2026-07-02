# ADR 0015 - Use Transactional Outbox For Operational Emails

Status: Accepted

Date: 2026-07-02

## Context

The application sends operational emails for:

- booking confirmations
- booking updates
- booking cancellations
- email verification
- password reset
- MFA login codes

Originally, some flows sent email inline during the request.

That creates a reliability problem: a user action can succeed in the database but appear to fail because the email provider is unavailable.

## Decision

Use a transactional outbox table for operational emails.

Domain mutations write an `OutboxEvent` row in the same database transaction as the business change.

A scheduled backend processor reads due events, sends the email through Resend, and marks the event as processed or failed with retry metadata.

## Alternatives Considered

## Send Email Inline During Requests

Pros:

- simpler code path
- immediate send attempt

Cons:

- external email failures can make successful user actions look failed
- no durable retry record
- harder to recover from temporary provider outages

## Add A Full Queue Immediately

Pros:

- dedicated background processing
- better throughput and retry tooling
- useful for larger workloads

Cons:

- extra infrastructure
- more operational complexity
- unnecessary for the current MVP email volume

## Decision Rationale

The transactional outbox gives the app the most important reliability property without adding a heavy queue yet.

If the booking or auth mutation commits, the email work is durably recorded.

If Resend is unavailable, the scheduled processor can retry later.

## Trade-Offs

Email delivery is no longer part of the request lifecycle, so emails may be sent shortly after the user action rather than synchronously.

The processor also needs monitoring so failed events do not go unnoticed.

## Consequences

Positive consequences:

- booking and auth mutations are not rolled back by email provider failures
- retry metadata is stored in the database
- email processing has one shared implementation
- the always-on NestJS API has a natural home for the processor

Negative consequences:

- emails are eventually sent rather than guaranteed immediate
- outbox rows need cleanup or retention rules later
- processor failures need logging and operational visibility

## Follow-Up Work

- add observability around failed outbox events
- decide retention rules for processed events
- consider a dedicated queue only if volume or worker isolation requires it
