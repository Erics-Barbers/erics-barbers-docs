# Local Development Setup

Status: current setup guide for the Next.js web client, NestJS API, and React Native/Expo mobile scaffold.

Related notes:

- [[Current System Architecture]]
- [[Authentication Flows]]
- [[Mobile App Delivery Roadmap]]
- [[ADR 0025 - Establish The NestJS OpenAPI Document As The Canonical Client Contract]]

## Repositories

The repositories are independent Git projects and should be installed and run from their own directories:

| Repository | Purpose |
| --- | --- |
| `erics-barber-api` | Shared NestJS backend and canonical OpenAPI contract |
| `erics-barbers-ui` | Next.js web client and browser authentication BFF |
| `erics-barbers-app` | React Native and Expo customer application |
| `erics-barbers-docs` | Requirements, architecture, roadmaps, and ADRs |

Examples below assume a shell opened in the workspace root. Adjust paths to match the local clone location.

## Shared Prerequisites

- Node.js 22; the API currently pins `22.21.0`
- npm
- Git
- PostgreSQL

Use the API's pinned Node version across the workspace unless a client repository later records a different supported version.

## API Setup

```bash
cd erics-barber-api
npm install
```

Create `.env` with local values. Variables currently used by the API include:

| Variable | Purpose |
| --- | --- |
| `DATABASE_URL` | PostgreSQL connection string used by Prisma. |
| `JWT_SECRET` | JWT signing and verification secret. |
| `RESEND_API_KEY` | Transactional email credentials. |
| `CLIENT_BASE_URL` | Customer web origin for CORS and customer email links. |
| `STAFF_CLIENT_BASE_URL` | Staff web origin for staff password-reset links. |
| `BOOKING_ENABLED` | Enables guarded booking endpoints when `true`. |
| `AUTH_EXTERNAL_PROVIDERS_ENABLED` | Enables external-provider foundations when `true`; providers are not implemented. |

Example:

```dotenv
DATABASE_URL="postgresql://postgres:password@localhost:5432/erics_barber"
JWT_SECRET="replace-with-a-local-development-secret"
RESEND_API_KEY="replace-with-a-development-key"
CLIENT_BASE_URL="http://localhost:3000"
STAFF_CLIENT_BASE_URL="http://staff.localhost:3000"
BOOKING_ENABLED="true"
AUTH_EXTERNAL_PROVIDERS_ENABLED="false"
```

Never commit real credentials.

Generate Prisma code, apply development migrations, and optionally seed booking data:

```bash
npx prisma generate
npx prisma migrate dev
npm run db:booking:seed
```

Start the API:

```bash
npm run start:dev
```

Local endpoints:

- API: `http://localhost:4000`
- Swagger UI: `http://localhost:4000/api`
- health: `http://localhost:4000/health`

Useful API commands:

| Command | Purpose |
| --- | --- |
| `npm run test` | Run unit tests. |
| `npm run test:e2e` | Run API end-to-end tests. |
| `npm run lint` | Run ESLint with fixes. |
| `npm run prisma:ui` | Open Prisma Studio. |
| `npm run db:booking:reset` | Remove only the deterministic demo booking data. |
| `npm run db:booking:reseed` | Reset and recreate demo booking data. |
| `npm run openapi:generate` | Regenerate the canonical OpenAPI document. |
| `npm run openapi:check` | Fail if the committed OpenAPI document is stale. |

Installing dependencies activates the tracked Husky pre-commit hook. API commits run `openapi:check` and stop if controller/DTO metadata no longer matches `openapi/openapi.json`. CI must also run this check because local hooks can be skipped.

## Web Setup

```bash
cd erics-barbers-ui
npm install
```

Create `.env.local` as needed:

```dotenv
NEXT_PUBLIC_API_BASE_URL="http://localhost:4000"
NEXT_PUBLIC_BOOKING_ENABLED="true"
NEXT_PUBLIC_AUTH_EXTERNAL_PROVIDERS_ENABLED="false"
NEXT_PUBLIC_STAFF_SITE_URL="http://staff.localhost:3000"
NEXT_PUBLIC_TEST_STAFF_SITE_URL="http://staff.test.localhost:3000"
```

Start the web application:

```bash
npm run dev
```

The customer site normally runs at `http://localhost:3000`. Host-aware staff and test routing may require using the configured local hostnames.

Browser authentication goes through Next.js BFF routes under `/api/auth/*`. Access and refresh tokens are stored in HttpOnly cookies on the web domain. Do not call NestJS auth endpoints directly from browser UI code.

## Mobile Setup

In addition to the shared prerequisites, install:

### iOS on macOS

- Xcode and Xcode Command Line Tools
- an iOS Simulator runtime
- CocoaPods, when required by native dependency installation

### Android

- Android Studio
- Android SDK and platform tools
- Java 17
- a configured Android Virtual Device, or a physical device with USB debugging
- `ANDROID_HOME` pointing to the SDK, or `android/local.properties` with a valid `sdk.dir` in a generated native project

Install the mobile dependencies:

```bash
cd erics-barbers-app
npm install
```

Build and install the native development client on a running simulator/emulator:

```bash
npm run ios
npm run android
```

Then start Metro for that development client:

```bash
npx expo start --dev-client
```

`npm run ios` and `npm run android` use Expo prebuild as needed. The generated `ios` and `android` folders are local build artifacts and remain ignored under the accepted CNG architecture. Native settings must be reproducible from `app.json` and config plugins.

The current mobile repository is a scaffold. Its API environment convention, generated API layer, TanStack Query setup, and native authentication storage are Mobile 1.0 delivery work; do not invent local environment variables or treat the browser cookie flow as its authentication contract.

### Emulator access to the local API

- iOS Simulator can normally reach the Mac API through `http://localhost:4000`.
- Android Emulator normally reaches the host machine through `http://10.0.2.2:4000`.
- A physical device needs a reachable LAN or tunnel URL and matching API CORS/network configuration.

Do not hard-code a development URL into feature code. The environment strategy is part of the mobile architecture ticket.

## Canonical OpenAPI Workflow

The API repository owns:

```text
erics-barber-api/openapi/openapi.json
```

After an accepted API contract change:

```bash
cd erics-barber-api
npm run openapi:generate
npm run openapi:check
cp openapi/openapi.json ../erics-barbers-ui/api/api-spec.json

cd ../erics-barbers-ui
npm run generate:api-client
npm run build
```

Mobile client generation will consume the same API-owned artifact when its API layer is introduced. Client copies are synchronized artifacts, not independent sources of truth.

## Typical Integrated Workflow

1. Start PostgreSQL.
2. Start NestJS on port `4000`.
3. Start the client being developed:
   - Next.js on port `3000`; or
   - Metro on port `8081` with an installed mobile development build.
4. Run the checks relevant to the changed repository.
5. If the API contract changed, regenerate, synchronize, and verify affected clients.

## Troubleshooting

### Web cannot reach the API

Check that the API is running, `NEXT_PUBLIC_API_BASE_URL` points to it, and `CLIENT_BASE_URL` matches the browser origin allowed by CORS.

### Android cannot build

Confirm an emulator/device is running, `java -version` reports Java 17, `ANDROID_HOME` points to the Android SDK, and `adb devices` sees the target.

### iOS development build is missing

`npx expo start --dev-client` serves JavaScript to an already installed development build. Run `npm run ios` first whenever the compatible native application is not installed or native configuration/dependencies have changed.

### OpenAPI pre-commit check fails

Run:

```bash
npm run openapi:generate
git add openapi/openapi.json
```

Review the contract diff before committing again.

### Email links are wrong

Check `CLIENT_BASE_URL`, `STAFF_CLIENT_BASE_URL`, Resend configuration, and the email template/link generation. Universal/app-link routing is planned mobile work and must retain a safe web fallback.

## Before Review

Run proportionate checks in every changed repository. At minimum:

```bash
# API
npm run openapi:check
npm run test

# Web
npm run lint
npm run build

# Mobile
npm run lint
```
