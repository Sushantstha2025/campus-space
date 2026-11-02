# CampusSpace — Project Context

> Concise working reference for agents. This is **not** a replacement for `docs/master-project-brief.md` (canonical specification) or `docs/decisions.md` (decision status and approvals).
>
> Last updated: 2026-11-02
>
> Status labels: **[ESTABLISHED]** supplied/approved requirement; **[TARGET]** planned architecture not yet built; **[PENDING]** unresolved and awaiting developer approval.

## Project overview

**[ESTABLISHED]** CampusSpace is a modular college management and concurrency-safe resource reservation system, built as a final-year software engineering project for Herald College Kathmandu.

It centralises selected college information and provides reliable booking of rooms, equipment and quantity-based inventory. Correctness, security, maintainability and demonstrable evaluation take priority over generating large amounts of code quickly.

**[ESTABLISHED]** Planning baseline: 24 October 2026. Four-month target; exact submission date to be confirmed. Solo developer on Windows, approximately 1–2 hours per day. Free-tier services only initially.

## Main modules

1. Resource catalogue and availability management.
2. Multi-resource reservation requests, approval and cancellation.
3. PostgreSQL-backed concurrency protection for unique and quantity resources.
4. Role-based access control.
5. Teacher availability and office hours.
6. Syllabus/course milestones.
7. College Calendar (college holidays, closures, events, important dates, announcements).
8. Resource Calendar (resource availability and bookings) — a separate module from the College Calendar.
9. Email notifications via a database-backed outbox/queue and Nodemailer.
10. Voice-assisted booking (later phase; browser Web Speech API only, manual booking always available).
11. Next.js web application.
12. Expo/React Native mobile application.
13. Repeatable functional, security, concurrency and performance testing for academic evidence.

## Approved technology baseline

**[ESTABLISHED]**

| Area | Choice |
|---|---|
| Repository | One Git monorepo; npm workspaces preferred |
| Web | Next.js + React + TypeScript |
| Mobile | Expo + React Native + TypeScript |
| Backend | Node.js + Express + TypeScript |
| Database | PostgreSQL + Prisma ORM |
| PostgreSQL-specific features | Reviewed raw SQL migrations or carefully isolated SQL |
| Email | Nodemailer through an SMTP provider (provider TBD) |
| Unit tests | Vitest |
| API integration tests | Supertest |
| Concurrency tests | Real dedicated PostgreSQL test database |
| Web E2E | Playwright (once the web app exists) |
| Voice | Web Speech API in a later phase; manual booking remains available |

**[PENDING]** Exact compatible versions of Node.js, TypeScript, Express, Next.js, Expo, Prisma and PostgreSQL (to be recommended from official documentation in Stage 02). Deployment provider. SMTP provider. Whether an OpenAPI specification is adopted.

## Target repository architecture

**[TARGET]** This is the agreed target, not permission to create every package before it is needed. Propose the minimum useful setup and explain any deviation.

```text
campusspace/
  apps/
    api/                 # Express + TypeScript
    web/                 # Next.js + TypeScript
    mobile/              # Expo + React Native + TypeScript
  packages/
    api-client/          # shared typed API client, if useful
    shared-types/        # safe shared contracts
    validation/          # shared schemas only where appropriate
  docs/
  tests/
    concurrency/
  package.json
  README.md
```

**[ESTABLISHED]** One root Git repository only; no nested repositories.

## User roles

**[ESTABLISHED]**

- **Student:** view the College Calendar and permitted Resource Calendar data; request resources; view and manage their own requests according to policy.
- **Teacher:** view the College Calendar; manage only the teacher information and academic data authorised for that account. Resource Calendar access is **[PENDING]** (no access vs. limited availability-only view).
- **Department / Resource Manager:** manage resources assigned to their department and review relevant requests.
- **System Admin:** manage system-level settings, users/roles and college-wide data.

**Registration and privilege rules [ESTABLISHED]:**

- Public registration permits Student or Teacher only.
- Department access must be granted or approved through a controlled admin process.
- System Admin accounts are created only through a deliberate seed/initial setup procedure using environment-supplied credentials, never hardcoded.
- Hiding a UI control is not authorization; the backend enforces every permission.

## Backend responsibilities

**[ESTABLISHED]**

- Own authentication, authorization and object-level ownership/department checks.
- Own validation, business rules and consistent error handling.
- Own reservation state transitions, approvals, cancellation and expiry.
- Enforce atomic multi-resource allocation and concurrency safety through PostgreSQL.
- Provide availability queries that are informative, not authoritative booking guarantees.
- Keep private booking details out of availability and calendar responses.
- Create notification intents transactionally and deliver email outside the booking transaction.
- Expose a documented API used consistently by web and mobile clients.
- Store timestamps in UTC and apply the documented college-local timezone for business rules/display.

## Database responsibilities

**[ESTABLISHED]**

- PostgreSQL is the final authority for reservation integrity, not frontend state or application-level pre-checks.
- Prevent overlapping confirmed/active allocations for unique resources at the database level where suitable (range/exclusion constraints with correct status handling).
- Protect quantity-based inventory against overallocation with a documented transaction/locking/serialization strategy; do not assume an exclusion constraint covers aggregate quantity.
- Multi-resource reservations are atomic: all required allocations commit or none.
- Define exactly which reservation states consume capacity; cancelled, rejected and expired requests release it.
- Record notification outbox intent in the same transaction as the booking/calendar state change.
- Preserve audit events for important state changes where justified.
- Maintain Prisma migration history as the schema source of change; document any raw SQL migration and keep it repeatable.
- Do not run destructive migrations against real data.

## Critical business rules

**[ESTABLISHED]**

- Every reservation requires department approval under the initial policy.
- A request must not be represented as confirmed until the required approval and final allocation transaction have succeeded.
- Multi-resource bookings must not partially confirm.
- Managers may approve only resources belonging to their department.
- A student's booking details must not leak unnecessarily through calendars or availability endpoints.
- Availability views do not bypass transactional checks on submission.
- Voice input, if added later, can only produce an editable draft; the user must review and submit the normal reservation flow, and the backend validates everything.
- Notification failure must not roll back a valid reservation.

**[PENDING]**

- Whether a pending request holds capacity until approval (recommended default: hold with expiry/release rules) — D-001.
- Multi-department approval aggregation (recommended default: each department approves its own items; overall approval only after all approve) — D-002.
- Teacher Resource Calendar access — D-003.
- College Calendar public vs. registered-user visibility — D-004.
- Email verification at registration — D-005.
- Department user model (one vs. multiple departments per person) — D-006.
- College timezone, opening hours, min/max booking duration, booking horizon and adjacent-interval convention — D-007.

## Security requirements

**[ESTABLISHED]**

- Backend authorization on every protected operation, including role, ownership and department/resource assignment.
- Public registration cannot grant Department or System Admin privileges.
- Passwords hashed with Argon2id or appropriately configured bcrypt.
- Short-lived access JWTs; rotating, revocable refresh tokens stored as hashes server-side.
- Web refresh token in a Secure, HttpOnly, SameSite cookie with CSRF/origin protection; access tokens in memory where practical.
- Mobile refresh credentials in Expo SecureStore; no long-lived secrets in plain AsyncStorage.
- Logout revokes the server-side session.
- Rate limits and safe error messages on authentication endpoints.
- Never log or commit passwords, tokens, cookies, secrets, SMTP credentials or real user data.
- Do not expose raw database errors, stack traces, SQL or internal details.
- Validate and authorize all input; do not trust client-side role or availability claims.

## Testing strategy

**[ESTABLISHED]**

- Vitest unit tests for business rules and validation.
- Supertest integration tests against the real API and persistence layer.
- Real dedicated PostgreSQL database for transaction and concurrency tests.
- No mocks as evidence that real API/database integration works.
- Concurrency harness under `tests/concurrency/` covering conflicting unique-resource bookings, adjacent intervals, quantity overallocation, atomic multi-resource failure, cross-department approvals and capacity release.
- Playwright for critical web journeys once the web app exists.
- Focused mobile verification on device/emulator; document unavailable platform tests honestly.
- Test role boundaries, ownership/IDOR, invalid input, booking conflicts, quantity limits, atomicity, notification failure/retry and auth/session revocation.
- Record actual workload, request counts, outcomes, retries, PostgreSQL errors and latency where measured. Never fabricate results.
- Distinguish passed, failed, not run and blocked.

## Development constraints

**[ESTABLISHED]**

- Solo developer, Windows, approximately 1–2 hours per day, four-month target.
- Free-tier services only initially; deployment provider selected later.
- Prefer a reliable, explainable, testable implementation over unnecessary abstraction or microservices.
- Project report and final evaluation are part of the workload.
- The voice assistant is an extended objective with a clear fallback to defer it if core integrity or the deadline is at risk.
- If time slips, protect in order: authentication/authorization, atomic reservations, real PostgreSQL concurrency tests, usable web core flows, usable mobile core flows, teacher/syllabus/calendar completeness, notification resilience, voice/visual polish.
- Do not begin a stage before its dependencies are accepted.

## Established vs. unresolved — quick view

**[ESTABLISHED]:** project purpose, modules, roles, technology baseline, monorepo target, reservation/approval policy, PostgreSQL concurrency authority, outbox notifications, security baseline, testing strategy.

**[PENDING]:** D-001 to D-010 in `docs/decisions.md`. These must not be treated as decided. The master brief's recommendations are starting points only.
