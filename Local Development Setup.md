# Local Development Setup

Status: current setup guide for the active Next.js frontend and NestJS backend.

This note explains how to run Eric's Barbers locally.

Related notes:

- [[Project Overview]]
- [[Current System Architecture]]
- [[Authentication Flows]]
- [[Database Design]]

## Active Projects

The active application is split across two folders:

| Folder | Purpose |
| --- | --- |
| `erics-barbers-ui` | Next.js frontend |
| `erics-barber-api` | NestJS backend API |

There is also an older or experimental Vite app:

`erics-barbers-ui-react`

That app does not appear to be part of the current active implementation.

## Prerequisites

Install:

- Node.js
- npm
- PostgreSQL
- access to a Resend API key if testing email sending

The backend `package.json` specifies:

```json
{
  "engines": {
    "node": "22.21.0"
  }
}
```

The README says Node 18 or higher, but the package metadata is more specific. Prefer Node 22 when working on this backend.

## Backend Setup

Open a terminal in:

```powershell
C:\Users\fahmi\Repositories\erics-barbers\erics-barber-api
```

Install dependencies:

```powershell
npm install
```

Create a `.env` file in `erics-barber-api`.

Verified backend environment variables:

| Variable | Purpose |
| --- | --- |
| `DATABASE_URL` | PostgreSQL connection string used by Prisma. |
| `JWT_SECRET` | Secret used to sign and verify JWTs. |
| `RESEND_API_KEY` | API key for sending transactional email through Resend. |
| `CLIENT_BASE_URL` | Frontend URL used for CORS and email verification/reset links. |
| `BOOKING_ENABLED` | Enables booking endpoints when set to `true`. |

Example local `.env`:

```dotenv
DATABASE_URL="postgresql://postgres:password@localhost:5432/erics_barber"
JWT_SECRET="replace-with-a-local-development-secret"
RESEND_API_KEY="replace-with-resend-key"
CLIENT_BASE_URL="http://localhost:3000"
BOOKING_ENABLED="false"
```

Do not commit real secrets.

## Database Setup

The Prisma schema is:

```powershell
prisma\schema.prisma
```

Prisma config is:

```powershell
prisma.config.ts
```

Generate the Prisma client:

```powershell
npx prisma generate
```

Apply migrations locally:

```powershell
npx prisma migrate dev
```

Open Prisma Studio if needed:

```powershell
npm run prisma:ui
```

## Running The Backend

Start the backend in development mode:

```powershell
npm run start:dev
```

The API listens on:

```text
http://localhost:4000
```

Swagger docs are available at:

```text
http://localhost:4000/api
```

Health check:

```text
http://localhost:4000/health
```

## Backend Commands

| Command | Purpose |
| --- | --- |
| `npm run start:dev` | Start NestJS in watch mode. |
| `npm run build` | Run Prisma migrate/generate and build NestJS. |
| `npm run test` | Run unit tests. |
| `npm run test:e2e` | Run e2e tests. |
| `npm run lint` | Run ESLint with fixes. |
| `npm run format` | Format TypeScript files. |
| `npm run format:prisma` | Format Prisma schema. |
| `npm run prisma:ui` | Open Prisma Studio. |

## Frontend Setup

Open a terminal in:

```powershell
C:\Users\fahmi\Repositories\erics-barbers\erics-barbers-ui
```

Install dependencies:

```powershell
npm install
```

Create a `.env.local` file in `erics-barbers-ui`.

Verified frontend environment variables:

| Variable | Purpose |
| --- | --- |
| `NEXT_PUBLIC_API_BASE_URL` | Base URL used by Next.js auth route handlers to call the backend. |
| `NEXT_PUBLIC_BOOKING_ENABLED` | Enables the booking page UI when set to `true`. |

Example local `.env.local`:

```dotenv
NEXT_PUBLIC_API_BASE_URL="http://localhost:4000"
NEXT_PUBLIC_BOOKING_ENABLED="false"
```

## Running The Frontend

Start the frontend in development mode:

```powershell
npm run dev
```

The frontend usually runs at:

```text
http://localhost:3000
```

## Frontend Commands

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start Next.js development server. |
| `npm run build` | Generate API client and build the Next.js app. |
| `npm run start` | Start the built Next.js app. |
| `npm run lint` | Run ESLint. |
| `npm run format` | Format app, test, and API files. |
| `npm run generate:api-client` | Generate the OpenAPI client into `api/generated`. |
| `npm run generate:api-types` | Generate OpenAPI TypeScript types. |

## Typical Local Workflow

1. Start PostgreSQL.
2. Start the backend:

```powershell
cd C:\Users\fahmi\Repositories\erics-barbers\erics-barber-api
npm run start:dev
```

3. Start the frontend:

```powershell
cd C:\Users\fahmi\Repositories\erics-barbers\erics-barbers-ui
npm run dev
```

4. Open the frontend:

```text
http://localhost:3000
```

5. Open Swagger if checking backend endpoints:

```text
http://localhost:4000/api
```

## API Client Generation

The frontend has a generated OpenAPI client in:

```powershell
api\generated
```

The source OpenAPI spec in the frontend repo is:

```powershell
api\api-spec.json
```

Generate the client:

```powershell
npm run generate:api-client
```

Important note:

The generated client is based on `api/api-spec.json`. If the backend API changes, the OpenAPI spec in the frontend repo must be updated before regenerating the client.

## Auth Development Notes

Authentication uses cookies and JWTs.

Frontend:

- stores `accessToken` as an HttpOnly cookie from the Next.js login route
- protects `/my-account` through `proxy.ts`

Backend:

- sets `refreshToken` as an HttpOnly cookie
- verifies Bearer access tokens through `AuthGuard`
- stores refresh-token sessions in PostgreSQL

Cookie note:

Some cookies are configured with `secure: true`. This is correct for production HTTPS, but if cookies do not appear during local HTTP development, cookie security settings should be the first thing to inspect.

## Booking Feature Flag

Booking has both frontend and backend feature flags.

Frontend:

```dotenv
NEXT_PUBLIC_BOOKING_ENABLED="false"
```

Backend:

```dotenv
BOOKING_ENABLED="false"
```

If the frontend flag is false, the booking page shows a "coming soon" message.

If the backend flag is false, the booking guard rejects booking API requests.

## Troubleshooting

## Frontend Cannot Reach Backend

Check:

- backend is running on port `4000`
- `NEXT_PUBLIC_API_BASE_URL` points to the backend
- backend CORS allows `CLIENT_BASE_URL`

## Verification Email Link Is Wrong

Check:

- backend `CLIENT_BASE_URL`
- Resend API key
- email content generated in `auth.prisma-repository.ts`

## Login Works But Protected Page Redirects

Check:

- `accessToken` cookie exists on the frontend domain
- token is not expired
- cookie `secure` behavior in local development
- `proxy.ts` matcher includes the route being accessed

## Backend Cannot Connect To Database

Check:

- PostgreSQL is running
- `DATABASE_URL` is correct
- migrations have been applied
- Prisma client has been generated

## Emails Do Not Send

Check:

- `RESEND_API_KEY`
- sender domain in `resend.service.ts`
- Resend account/domain configuration

## Before Opening A Pull Request

Recommended local checks:

Backend:

```powershell
npm run test
npm run lint
```

Frontend:

```powershell
npm run lint
npm run build
```

Run only the checks relevant to the files changed if time is limited, but the full checks are preferred before merging.

