# CampusSpace Backend Agent Instructions (`apps/api`)

Read the root `AGENTS.md`, `docs/agent-context/*`, `docs/decisions.md` and the master project brief before acting. This file adds backend-specific rules; it does not replace the canonical brief. Do not introduce technologies or architectural changes that the master brief and decision register do not approve.

## Approved stack

- Node.js + Express + TypeScript.
- PostgreSQL + Prisma ORM.
- Prisma migrations for normal schema evolution; reviewed raw SQL migrations only for PostgreSQL guarantees that Prisma cannot express.
- Vitest for unit tests, Supertest for API integration tests, and a real dedicated PostgreSQL database for transaction and concurrency tests.
- Do not add alternative frameworks, ORMs, databases, queues or validation libraries without approval.

## Modularity and separation of responsibilities

- Organise by domain module (for example: auth, users, resources, reservations, calendars, notifications). Keep modules cohesive and independently understandable.
- Keep HTTP concerns in routes/controllers, business rules in services, and persistence in repositories or a focused Prisma access layer.
- Controllers must not become large business-logic files.
- Make transaction boundaries explicit and own them in the service layer.
- Put cross-cutting concerns (configuration, errors, validation, logging, auth) in small, focused modules.
- Prefer a reliable, explainable, testable implementation over unnecessary abstraction. Do not introduce microservices.

## Validation conventions

- Validate all external input at the boundary: body, route params, query, headers and cookies.
- Use one approved schema-validation approach and avoid duplicating the same validation between layers.
- Validate types, ranges, enums, identifiers, date/time ranges, quantities, pagination values and unknown-field handling.
- Reject malformed input with a consistent 4xx response shape.
- Never trust client-side availability, role claims or permission hints.

## Centralised error handling

- Use central error-handling middleware and a consistent response/error shape with stable error codes.
- Never expose stack traces, internal SQL, file paths, secrets or raw database errors to clients.
- Map known domain and database errors deliberately (conflict, unique/exclusion violation, serialization failure, deadlock) to appropriate statuses.
- Log server errors with correlation/request IDs; log client errors at an appropriate level without noise.

## Authentication and authorization

- Public registration permits Student and Teacher only. A user must never self-assign Department/Resource Manager or System Admin privileges through a request body.
- Hash passwords with Argon2id or appropriately configured bcrypt.
- Use short-lived access JWTs and rotating, revocable refresh tokens.
- Store refresh tokens server-side as hashes and rotate them on refresh.
- Web refresh credentials use a Secure, HttpOnly cookie with deliberate CSRF/origin protection. Mobile refresh credentials use Expo SecureStore. Keep access tokens in memory where practical, never in browser localStorage.
- Deny access by default. Enforce role, ownership and department/resource-assignment checks in the backend for every protected operation, including object-level access (IDOR).
- Never log passwords, access tokens, raw refresh tokens, cookies or secrets.
- Apply rate limits and safe error messages to authentication endpoints.
- Logout must revoke the server-side session/refresh token.

## Database transaction requirements

- Use explicit database transactions for multi-step writes.
- A multi-resource reservation must commit all required allocations or none. No partial confirmation may survive a failure.
- Define and document the isolation level used per operation.
- Create notification outbox records in the same transaction as the relevant booking or calendar state change.
- Keep external I/O such as email delivery and network calls outside the critical transaction.
- Do not rely on an application-level pre-check for integrity.

## Reservation concurrency requirements

PostgreSQL is the final authority for reservation integrity.

- Unique resources (for example rooms) must not have overlapping confirmed/active allocations. Enforce this at the database level where suitable, using a range/exclusion constraint with correct status handling.
- Quantity-based resources must not be overallocated under concurrent requests. Use a transaction plus an explicit locking or serialization strategy and document exactly which rows are locked and why.
- Do not assume an exclusion constraint protects aggregate quantity.
- Define exactly which reservation states consume capacity. Cancelled, rejected and expired requests must release capacity.
- Use bounded retries only for recognised serialization failures or deadlocks. Never retry every error blindly. Document why retries are safe and observable.
- Handle deadlocks, unique violations and exclusion conflicts intentionally. Do not collapse every database error into a generic "unavailable" message.
- Add idempotency protection where justified to prevent duplicate submissions from client retries.
- Never treat a frontend availability check as a concurrency safeguard.

## Secure logging

- Use structured logs with request/correlation IDs.
- Redact passwords, tokens, cookies, secrets, authentication headers and unnecessary personal or booking data.
- Do not log full request bodies that may contain sensitive data.
- Log important state changes and security-relevant denials without leaking private information.
- Redact notification payloads and SMTP errors where required.

## Unit and integration testing

- Unit-test pure business rules, validation and state transitions.
- Integration-test routes with Supertest against a real PostgreSQL database, not a mock persistence layer.
- Cover authentication, authorization, role boundaries, ownership/IDOR attempts, department isolation, invalid input, pagination/filtering and date-range boundaries.
- Use a dedicated test database and document how it is provisioned. Never run tests against production or real user data.
- Keep tests deterministic and independent; clean up test data safely.

## PostgreSQL-backed concurrency testing

- Put repeatable concurrency tests under `tests/concurrency/`.
- Use real PostgreSQL behaviour, a dedicated test database and documented connection/setup settings.
- Cover at minimum: simultaneous conflicting room bookings, adjacent non-overlapping ranges, concurrent quantity requests against limited stock, atomic multi-resource failure, cross-department approval invalid states, and capacity release on cancel/reject/expire.
- Record workload, request count, success/failure outcomes, retry counts, relevant PostgreSQL errors and latency percentiles where measured.
- Never claim a concurrency or performance result without actual measurements.

## Migration safety

- Prisma migration history is the source of schema change. Do not edit an already-applied migration; add a new one.
- Use reviewed raw SQL migrations only for PostgreSQL-specific features, and document them so they are repeatable.
- Never run destructive migrations against real or production data without explicit approval and a backup plan.
- Define foreign keys, uniqueness, deletion rules, indexes, timestamps and relevant check constraints deliberately.
- Seed only safe development data. Admin credentials must come from environment variables or an explicit secure setup process, never source-code literals.
- Document migration and seed commands before running them.

## API documentation conventions

- Document every endpoint: method and path, authentication requirements, roles and object-level permissions, request body/query parameters, successful response, validation/error responses, pagination/filtering, date/time formats, token-refresh behaviour and relevant state transitions.
- Keep a consistent response/error shape and API versioning convention.
- Keep OpenAPI or contract documentation synchronised with the actual implementation and real DTOs.
- Store timestamps in UTC and apply the documented college-local timezone for display and business rules.
- Do not change business logic merely to simplify a UI without recording the decision and updating tests.
