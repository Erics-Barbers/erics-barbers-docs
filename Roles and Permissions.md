# Roles and Permissions

Status: planned authorization model for current and upcoming features.

This note defines the application's user roles and the intended permission boundaries between customers, barbers, and admins.

Related notes:

- [[Project Overview]]
- [[Database Design]]
- [[Authentication Flows]]
- [[Known Gaps and Roadmap]]

## Current Model

Every person who can log in is represented by a `User` record.

The `User.role` enum identifies the account's authorization role:

- `CUSTOMER`
- `BARBER`
- `ADMIN`

The schema can store these roles today, but role-based enforcement is not complete across all unfinished modules. Backend route guards must still enforce these permissions as barber, booking, and admin workflows are built.

## Role Definitions

## Customer

A customer is a public user who books and manages their own appointments.

Intended permissions:

- register for an account
- verify their email address
- log in and manage their own profile
- view available services
- create bookings for themselves
- view their own bookings
- cancel or reschedule their own bookings, subject to shop policy

Customers must not be able to:

- view another customer's bookings or profile data
- manage barber availability
- create or deactivate barber records
- manage services, pricing, or operational settings
- view all shop bookings unless the booking belongs to them

## Barber

A barber is a staff user who can receive bookings and manage their working schedule.

A barber should be represented by both:

- a `User` record with `role = BARBER`
- a linked `Barber` profile record containing barber-specific data, such as display name, active state, and phone number

Intended permissions:

- log in to the staff-facing area
- view their own dashboard
- view bookings assigned to them
- view relevant customer appointment details for assigned bookings
- manage their own availability, once availability is modelled
- update operational booking state where the change is part of the barber workflow, such as marking an appointment complete

Barbers must not be able to:

- manage another barber's availability unless explicitly promoted to admin
- create, deactivate, or delete other barber accounts
- manage global services or pricing
- view unrelated customer data
- perform admin-only operational actions

## Admin

An admin is an operational owner or manager of the application.

Intended permissions:

- log in to the admin or staff-facing area
- view and manage all bookings
- create, onboard, update, deactivate, or remove barber profiles
- assign or reassign bookings to barbers
- manage services, prices, durations, and availability rules once those models exist
- handle customer support actions such as cancellation or rescheduling
- view operational reports or audit logs once implemented

Admins should still be subject to authentication, MFA, audit logging for sensitive actions, and least-privilege checks inside the backend.

## Staff Area

The staff-facing product surface may live under route prefixes such as:

- `/barber`
- `/dashboard`
- `/calendar`
- `/settings`
- `/admin`

If a staff subdomain is used, such as `staff.example.com`, the Next.js app can route that host to the staff area while keeping the same backend role checks.

The subdomain improves navigation and product separation, but it does not replace authorization. The backend must still verify the authenticated user's role for every protected staff or admin API operation.

## Enforcement Rules

Authorization should be enforced in the backend, not only in the frontend.

Frontend checks can hide navigation and redirect users, but they are not security boundaries. The NestJS API should enforce permissions through guards and use-case checks before reading or changing protected data.

Recommended rules:

- Customer routes require `CUSTOMER`, `BARBER`, or `ADMIN` only when the action is allowed for that role.
- Barber routes require `BARBER` or `ADMIN`.
- Admin routes require `ADMIN`.
- Ownership checks are still required when a user can only access their own resource.
- Role checks and ownership checks should both be tested.

## Minimum Permission Matrix

| Capability | Customer | Barber | Admin |
| --- | --- | --- | --- |
| Manage own profile | Yes | Yes | Yes |
| Create own booking | Yes | Optional | Yes |
| View own bookings | Yes | Yes, for assigned bookings | Yes |
| View all bookings | No | No | Yes |
| Manage own availability | No | Yes | Yes |
| Manage another barber's availability | No | No | Yes |
| Manage services and pricing | No | No | Yes |
| Create or deactivate barber profiles | No | No | Yes |
| Access staff dashboard | No | Yes | Yes |
| Access admin dashboard | No | No | Yes |

## Implementation Notes

The backend already has a `Role` enum and role-decorator foundation. As role enforcement is completed:

1. Include role information in authenticated request context.
2. Apply role guards consistently to protected controllers.
3. Add ownership checks inside use cases for user-owned resources.
4. Keep barber-specific checks aligned with the linked `Barber` profile.
5. Add tests for allowed, forbidden, and not-found cases.

## Mental Model

The role model should stay simple:

1. Everyone who logs in is a `User`.
2. Customers manage their own appointment journey.
3. Barbers manage their assigned working day.
4. Admins manage shop operations.
5. The frontend may route users differently, but the backend owns authorization.
