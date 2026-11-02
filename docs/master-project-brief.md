# CampusSpace --- AI-Agent Prompt Pack

## Four-month, backend-first development plan

**Project:** CampusSpace --- A Modular College Management &
Concurrency-Safe Resource Reservation System\
**Context:** Final-year project for Herald College Kathmandu\
**Planning baseline:** 24 October 2026; four-month target, exact
submission date to be confirmed\
**Developer:** Solo developer, Windows, approximately 1--2 hours per
day\
**Repository strategy:** One Git monorepo\
**Web:** Next.js + React + TypeScript\
**Mobile:** Expo + React Native + TypeScript\
**Backend:** Node.js + Express + TypeScript\
**Database:** PostgreSQL + Prisma ORM, with explicit SQL/migrations
where PostgreSQL-specific guarantees are needed\
**AI voice feature:** Later phase; Web Speech API for browser speech
recognition, with manual booking always available

------------------------------------------------------------------------

# Part A --- How to use this pack

1.  Create the main `campusspace` Git repository.
2.  Give the **Master Project Brief** to your primary coding agent.
3.  Run prompts in the order shown in Part C. Do not ask an agent to
    implement a later stage before its dependencies are accepted.
4.  Store the canonical documents in `docs/`. Agents must read them
    before modifying code.
5.  Give one agent ownership of each task at a time. Other agents can
    review, but should not independently modify the same files.
6.  After each stage, inspect the diff, run the stated checks, commit
    working code, and update the handoff report.
7.  Do not allow an agent to claim tests passed unless it actually ran
    them and reports the result.
8.  If a decision changes, update `docs/decisions.md` first and assess
    its effect on the API, schema, tests, web app, mobile app, and
    report.

## Recommended use of the available tools

-   **Primary coding agent:** Choose either Codex or OpenCode for
    implementation ownership. Use whichever has reliable access to your
    local repository and can run terminal commands. Avoid switching
    primary agents mid-task unless necessary.
-   **Secondary code reviewer:** GitHub Copilot, Codex, Gemini, or
    another available agent can review a specific diff and test output.
    Ask for findings, not an independent rewrite.
-   **Planning/research:** Gemini or ChatGPT can challenge requirements,
    review designs, and help explain technical choices. Verify any
    version-specific claims against official documentation.
-   **UI design:** Stitch creates design concepts and screens. Its
    output is a design input, not authority to change backend
    requirements or security rules.
-   **Other agents (Antigravity, Free Buff, OpenCode, etc.):** Use them
    for bounded tasks only after confirming repository access, available
    capabilities, and file-editing permissions.

Never send all prompts at once. Each prompt is intended to be a discrete
task.

------------------------------------------------------------------------

# Part B --- Master Project Brief

Give the following prompt to the primary coding agent at the beginning
of the project.

## Prompt 00 --- Establish the project context

You are the primary engineering agent for CampusSpace, a final-year
software engineering project for Herald College Kathmandu. You must help
plan and implement the project incrementally, with correctness,
security, maintainability, and demonstrable evaluation taking priority
over generating large amounts of code quickly.

### Product goal

CampusSpace centralises selected college information and provides a
reliable resource-reservation system. It includes:

1.  Resource catalogue and availability management.
2.  Multi-resource reservation requests, approval and cancellation.
3.  PostgreSQL-backed concurrency protection.
4.  Role-based access control.
5.  Teacher availability and office hours.
6.  Syllabus/course milestones.
7.  A public College Calendar.
8.  A restricted Resource Calendar.
9.  Email notifications using a database-backed outbox/queue and a
    Node.js mailer such as Nodemailer.
10. A later-phase voice-assisted booking feature using browser speech
    recognition where supported.
11. Web application using Next.js and TypeScript.
12. Mobile application using Expo, React Native, and TypeScript.
13. Repeatable functional, security, concurrency, and performance tests.

The voice feature is not part of the initial core implementation. Manual
booking must remain usable if speech recognition or an optional parsing
service is unavailable.

### Required architecture

Use one Git repository, preferably an npm-workspaces monorepo:

``` text
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

This is a target structure, not permission to create every package
before it is needed. Propose the minimum useful setup and explain any
deviation. Use one root Git repository; do not create nested Git
repositories.

### Backend and database rules

-   Use Node.js, Express, TypeScript, PostgreSQL, and Prisma ORM.
-   Use Prisma migrations for normal schema evolution.
-   Use reviewed raw SQL migrations or carefully isolated SQL for
    PostgreSQL features that Prisma does not fully express, including
    exclusion constraints if appropriate.
-   PostgreSQL, not frontend state or an application-level pre-check, is
    the final authority for reservation integrity.
-   Multi-resource booking must be atomic: either all required
    allocations are committed or none are.
-   Unique resources such as rooms must not have overlapping
    confirmed/active allocations.
-   Quantity-based resources must not be overallocated when requests
    race.
-   Define the exact reservation states that consume capacity.
    Cancelled/rejected/expired requests must not continue consuming
    capacity.
-   Do not assume an exclusion constraint alone solves quantity-based
    inventory. Design and test a transaction/locking strategy for
    quantity allocations.
-   Handle transaction retries and recognised PostgreSQL conflict,
    serialization, and deadlock errors deliberately. Do not retry every
    error blindly.
-   Keep notification delivery outside the critical booking transaction
    using an outbox or durable queue record created transactionally with
    the booking state change.
-   Use UTC for stored timestamps and a documented college-local
    timezone for display and business rules. Confirm the intended
    timezone with the developer before implementing time-sensitive
    rules.

### Roles and permissions

Initial roles:

-   **Student:** may view the College Calendar and permitted Resource
    Calendar data, request resources, and view/manage their own requests
    according to policy.
-   **Teacher:** may view the College Calendar and manage only the
    teacher information and academic data authorised for that account.
-   **Department / Resource Manager:** may manage resources assigned to
    their department and review relevant requests.
-   **System Admin:** may manage system-level settings, users/roles, and
    college-wide data.

Public registration permits Student or Teacher registration only. A user
cannot grant themselves Department or System Admin privileges by
selecting a role in a request body. Department access must be
granted/approved through a controlled admin process. System Admin
creation must use a deliberate seed or initial setup procedure;
credentials must come from environment variables or an explicit secure
setup process, never source-code literals. Hash passwords with a modern
password-hashing algorithm such as Argon2id or appropriately configured
bcrypt.

The College Calendar is visible to all authenticated users (and may be
made publicly readable if the product owner confirms that policy).
Editing is restricted to System Admin and explicitly authorised
department users. The Resource Calendar is available to Students and
authorised Department users; other roles must not gain access by
guessing API URLs. Define whether teachers need any limited availability
view and ask the developer if this is ambiguous.

All reservations require department approval in the initial policy. A
request must not be represented as confirmed until the required approval
and final allocation transaction have succeeded. Decide and document
whether capacity is held while pending approval; the recommended default
is to reserve capacity transactionally while a request is pending, with
expiry/cancellation rules, to avoid approving more requests than
capacity allows. Confirm this policy with the developer before
finalising booking logic.

Authorization must be enforced in the backend for every protected
operation, including object-level ownership and department assignment.
Hiding a button is not authorization.

### Authentication baseline

Recommend short-lived access JWTs and rotating, revocable refresh tokens
because both browser and mobile clients are planned. Before
implementation, document the exact flow:

-   Access token lifetime initially around 10--15 minutes, subject to
    security review.
-   Refresh tokens stored server-side as hashed session/token records
    and rotated on refresh.
-   Web: use a Secure, HttpOnly, SameSite cookie for the refresh token;
    design CSRF protections and origin policy deliberately. Keep access
    tokens in memory where practical rather than localStorage.
-   Mobile: use Expo SecureStore for refresh credentials; keep access
    tokens in memory where practical.
-   Logout revokes the session/refresh token and clears client
    credentials.
-   Password reset, email verification, and account recovery must be
    scoped explicitly; do not invent a complete production identity
    system without discussing effort.
-   Never log passwords, raw refresh tokens, access tokens, cookies, or
    secrets.
-   Use rate limits and safe error messages for authentication
    endpoints.

If a safer or simpler approach is recommended after checking the current
framework and deployment constraints, explain the trade-offs and request
approval before changing this baseline.

### Calendars

**College Calendar:** college holidays, closures, events, important
dates, and general announcements. It is separate from resource
reservations. Editing: System Admin and explicitly authorised Department
users.

**Resource Calendar:** resource availability, bookings, and relevant
statuses for rooms/equipment/quantity-based items. It is available to
Students and authorised Department users according to permissions. Do
not leak private details about another user's booking; show only the
information required to communicate availability and operational status.

Use separate domain modules, API permissions, and UI navigation. Shared
date/time utilities are acceptable.

### Notifications

Use a database-backed outbox/queue. The booking transaction records the
notification intent. A background worker sends email through Nodemailer
and an SMTP provider. Delivery failures must not silently roll back a
valid reservation. Use retries with backoff, attempt counts, timestamps,
final failure status, and safe logs. The actual email provider and
credentials will be selected later; configuration must use environment
variables.

### Frontend

-   Web: Next.js + TypeScript, responsive and accessible.
-   Mobile: Expo + React Native + TypeScript, Android and iOS as target
    platforms.
-   Both clients use the same documented API and business rules.
-   Share API client code, types, and suitable validation where it
    reduces duplication, but do not force platform-specific UI into one
    component system.
-   Design will be explored in Stitch. Treat designs as references and
    preserve accessibility, responsive behaviour, and functional
    requirements.
-   Avoid building all screens before the API contract and key user
    journeys are understood.

### Testing and academic evidence

Use a practical layered testing strategy: - Vitest for unit tests. -
Supertest for Express API integration tests. - A real dedicated
PostgreSQL test database for transaction and concurrency tests. -
Playwright for critical web end-to-end journeys once the web app
exists. - Focused mobile verification on a device or emulator; document
any unavailable platform tests. - Test role boundaries, ownership,
invalid input, booking conflicts, quantity limits, atomic multi-resource
operations, notification failures, and authentication/session
revocation.

Record concurrency-test workload, request count, success/failure
outcomes, latency percentiles where practical, retries, and relevant
PostgreSQL errors. Do not claim a benchmark result without actual
measurements.

### Development constraints

-   Solo developer on Windows, approximately 1--2 hours per day,
    four-month target.
-   Free-tier services only initially.
-   Deployment provider will be selected later.
-   Use the Web Speech API in a later phase; browser support varies.
    Keep manual entry available.
-   Project report and final evaluation are part of the workload.
-   Prefer a reliable, explainable, testable implementation over
    unnecessary abstraction or microservices.

### Mandatory agent behaviour

1.  Start by reading all existing `docs/` files, `README.md`, repository
    structure, and relevant code.
2.  Do not overwrite or regenerate working files without inspecting
    them.
3.  Before a coding task, state a short plan, files expected to change,
    dependencies, and acceptance criteria.
4.  Implement only the requested stage. Do not quietly add unrelated
    features.
5.  If a requirement is ambiguous or a security/data-integrity decision
    is consequential, ask or propose a default and document it rather
    than guessing.
6.  Make small, reviewable changes.
7.  Run relevant format, lint, typecheck, migration, and test commands
    available in the project. Report commands and actual results.
8.  Never claim a test passed if it was not run. Distinguish "not run,"
    "blocked," and "failed."
9.  Update relevant documentation and `docs/decisions.md`.
10. At the end, provide a handoff containing: summary, files changed,
    commands/tests run and results, decisions made, known issues, and
    the next recommended task.
11. Do not expose secrets or commit `.env` files.
12. Never put real admin credentials, API keys, SMTP passwords, or token
    secrets in source code.
13. Do not use mock data as evidence that real API/database integration
    works.
14. Use official documentation to verify current APIs and versions when
    needed; avoid unnecessary dependency upgrades.
15. Do not begin the next stage until the developer has reviewed and
    accepted the current stage.

First, inspect the empty/new repository state and produce a proposed
implementation plan, risks, milestones, and initial decision list. Do
not generate the entire application in one response.

------------------------------------------------------------------------

# Part C --- Staged prompts

## Stage 01 --- Requirements and scope baseline

**Goal:** Convert the proposal and updated decisions into a testable
specification before implementation.

### Prompt

Read the Master Project Brief and the supplied CampusSpace proposal.
Create or update:

-   `docs/requirements.md`
-   `docs/architecture.md` (high-level draft only)
-   `docs/decisions.md`
-   `docs/roadmap.md`
-   `docs/roles-and-permissions.md`

Requirements: 1. Preserve the proposal's core scope: resource booking,
teacher availability, syllabus milestones, calendars, notifications,
concurrency-safe allocation, evaluation, and later voice-assisted
booking. 2. Incorporate the new separate College Calendar and Resource
Calendar. 3. List user stories and acceptance criteria for each role and
module. 4. Define reservation lifecycle states, including request,
pending approval, approved/confirmed, rejected, cancelled, expired if
applicable, and completed if applicable. Recommend a minimal state
machine and explain transitions. 5. Explicitly list unresolved questions
and recommended defaults. At minimum address pending-request capacity
holds, whether teachers can see the Resource Calendar, whether calendar
data is publicly readable, and how department/resource assignments are
administered. 6. Define in-scope, out-of-scope, and deferred features.
7. Create a dependency-aware four-month roadmap that reserves time for
report writing, testing, and final demonstration. 8. Do not code
application features yet.

Acceptance criteria: - Every feature has a testable outcome. - Role
permissions are explicit. - Calendar permissions are separate. -
Concurrency and quantity safety are identified as core research
requirements. - Unresolved decisions are visible, not hidden. - The
developer can approve the specification before database design begins.

Handoff: summarise decisions needing developer approval.

## Stage 02 --- Architecture and repository foundation

**Goal:** Create the monorepo and documented boundaries.

### Prompt

Read and follow the approved requirements and decisions. Set up the
minimum viable npm-workspaces monorepo:

-   `apps/api`
-   `apps/web`
-   `apps/mobile`
-   `packages/` only for shared code that has a clear use
-   `docs/`
-   `tests/concurrency/`

Tasks: 1. Recommend compatible stable versions of Node.js, TypeScript,
Express, Next.js, Expo, Prisma, and PostgreSQL based on official
documentation. Explain any compatibility constraint. 2. Create root
workspace scripts that make sense for this repository. 3. Add consistent
formatting/linting and TypeScript configuration without overengineering.
4. Create `.gitignore`, `.env.example` files, and a README with Windows
setup instructions. 5. Ensure secrets and local databases are not
committed. 6. Add a basic health endpoint in the API only if it is part
of the agreed initial skeleton. 7. Document how each app will run
locally. 8. Do not implement business features.

Acceptance criteria: - All apps have minimal, clearly separated
scaffolds. - Root commands are documented and run successfully. - No
nested Git repositories exist. - No real secrets are present. - The
README gives a reproducible setup path on Windows.

## Stage 03 --- Data model and PostgreSQL design

**Goal:** Agree on a schema before implementing endpoints.

### Prompt

Read requirements, roles/permissions, architecture, and decisions.
Design the PostgreSQL schema using Prisma for CampusSpace. Produce:

-   `docs/database-design.md`
-   Updated Prisma schema
-   Initial migration(s), after review
-   Seed strategy using safe development-only data
-   An entity relationship description or Mermaid ER diagram in the
    documentation

Consider models for: - User and role/role assignment - Department -
Resource and resource category/type - Unique resources such as rooms -
Quantity-based inventory and allocations - Reservation request and
reservation line/allocation - Approval records and audit events where
justified - Teacher availability/office hours - Course/module/syllabus
milestones - College Calendar events/closures/holidays - Notification
outbox and delivery attempts - Refresh-token sessions -
Resource/department ownership and assignment

Rules: 1. Do not create every possible table without explaining its
purpose. 2. Define foreign keys, uniqueness, deletion rules, indexes,
timestamps, and relevant check constraints. 3. Clearly distinguish a
reservation request from individual resource allocations. 4. Explain how
one reservation can request multiple resources atomically. 5. Decide how
quantity inventory is represented and how concurrent allocations are
protected. 6. Plan PostgreSQL range/exclusion constraints for
overlapping allocations of unique resources where suitable. Include
extension requirements if needed and check compatibility with the chosen
PostgreSQL version. 7. Do not assume an exclusion constraint protects
aggregate quantities. 8. Define which statuses consume capacity and how
cancellation/rejection/expiry releases it. 9. Define UTC storage and
college-local display rules. 10. Keep Prisma migration history as the
schema source of change. Any raw SQL migration must be documented and
repeatable. 11. Do not run destructive migrations against any real
database.

Acceptance criteria: - The schema supports all approved user stories. -
The design explains both unique-resource and quantity-resource
concurrency. - The developer approves the schema before endpoint
implementation. - Migration and seed instructions are documented.

## Stage 04 --- Backend foundation and API conventions

**Goal:** Establish a maintainable Express API before implementing
modules.

### Prompt

Using the approved schema and architecture, implement the backend
foundation in `apps/api`:

-   Express app and server entry points
-   TypeScript configuration
-   Prisma client lifecycle
-   Environment validation
-   Central error-handling middleware
-   Request validation convention
-   Structured, redacted logging
-   Health/readiness endpoints
-   CORS and security middleware configured from environment
-   API versioning convention, if chosen in the architecture
-   Unit-test and integration-test setup
-   Consistent response/error shape
-   API documentation or OpenAPI specification if approved

Choose a schema validation library (for example, Zod) only after
checking current compatibility and explaining its role. Do not duplicate
validation unnecessarily between layers.

Rules: - Controllers should not become giant business-logic files. - Do
not expose raw database errors, stack traces, secrets, or internal SQL
to clients. - Do not add business endpoints beyond the foundation. - Do
not disable TypeScript checks to make builds pass.

Acceptance criteria: - API starts from documented commands. -
Environment variables are validated on startup. - Health/readiness
behaviour is documented. - Error responses are consistent. - Typecheck,
lint, and foundational tests run successfully.

## Stage 05 --- Authentication and authorization

**Goal:** Implement identity and role boundaries before feature modules.

### Prompt

Implement registration, login, logout, refresh, and current-user
functionality using the approved authentication design.

Baseline: - Public registration allows Student or Teacher only. -
Department role requires admin approval or an administrator-controlled
invitation. - System Admin cannot be assigned through public
registration. - Initial admin is provisioned through a safe, explicit
setup/seed process using environment-supplied credentials or a
controlled setup mechanism. Never hardcode a password. - Passwords are
hashed using an approved secure password-hashing configuration. - Access
JWTs are short-lived. - Refresh tokens are rotated, stored as hashes
server-side, and revocable. - Browser refresh credentials use secure
HttpOnly cookies with CSRF/origin protections designed for the chosen
deployment. - Mobile refresh credentials use Expo SecureStore. - Never
store long-lived secrets in browser localStorage. - Logout revokes the
server-side session. - Add rate limiting and safe authentication error
messages.

Implement authorization middleware and reusable permission checks,
including: - role-level permissions - ownership checks -
department/resource assignment checks - rejection of self-assigned
privileged roles

Tests must cover: - valid/invalid login - password verification - token
expiry - refresh rotation and reuse/revocation behaviour - logout -
unauthorized roles - IDOR/object ownership attempts - department
assignment boundaries - public attempts to create Admin/Department
accounts

Acceptance criteria: - Both web and mobile auth flows are documented. -
Protected routes deny access by default. - Privilege escalation tests
fail safely. - Tests actually run against the relevant persistence
layer.

## Stage 06 --- Resource catalogue and availability

**Goal:** Let departments maintain their own resources safely.

### Prompt

Implement resource management APIs and business logic according to the
approved schema.

Requirements: - Department managers can create/update/disable resources
only within their assigned department. - Model unique resources (for
example, individual rooms) separately in behaviour from quantity-based
inventory (for example, pens or chart paper). - Store capacity, status,
category, location, operating constraints, and other fields only where
approved by requirements. - Define validation for quantities, date/time,
active/inactive resources, and department ownership. - Availability
queries are informative, not authoritative booking guarantees. - Do not
leak private reservation details through availability endpoints. - Add
filtering, pagination, and stable sorting where lists can grow. - Add
audit history for important resource changes if it is in approved scope.

Tests: - permissions and department isolation - inactive resources -
invalid quantities - availability edge cases - overlapping time
boundaries - pagination and filters

Acceptance criteria: - Only authorised managers can change their
resources. - Students can discover permitted resources. - Availability
results are consistent with the approved reservation-state policy. - All
tests pass or failures are reported truthfully.

## Stage 07 --- Reservation and concurrency engine

**Goal:** Build the most important technical component of CampusSpace.

### Prompt

Implement the reservation workflow with PostgreSQL as the final
authority. Before coding, explain the transaction strategy for unique
resources and quantity-based resources separately.

Functional requirements: - Student submits one request containing one or
more resources, dates/times, and quantities. - Every reservation
requires department approval under the current policy. - Each resource
line is routed to its responsible department. - Define how multiple
departments approve one reservation. A recommended default is that the
overall reservation becomes approved only when all required department
approvals are complete. - Define pending-request capacity holds with the
developer-approved policy. - Prevent partial confirmation: if any
required allocation fails, the transaction must not leave other
allocations committed. - Cancel/reject/expire reservations according to
state-transition rules and release capacity correctly. - Enforce
authorization for viewing, approving, rejecting, and cancelling. -
Prevent a manager from approving a resource belonging to another
department. - Preserve an audit trail of important state changes.

Concurrency requirements: 1. For unique resources, enforce no
overlapping active allocations at database level where suitable using a
range/exclusion constraint and correct status handling. 2. For
quantity-based resources, use a transaction and an appropriate
locking/serialization strategy so simultaneous requests cannot exceed
stock. Document exactly what rows are locked and why. 3. If serializable
isolation is used, implement bounded retries for known serialization
failures and document retry safety. 4. Handle deadlocks and
unique/exclusion conflicts intentionally; do not convert all database
errors into a generic "unavailable" message. 5. Use idempotency
protection where justified for client retries or duplicate submissions.
6. Create notification outbox records within the same transaction as the
relevant state change. 7. Do not trust a frontend availability check as
a concurrency safeguard.

Testing: - Two simultaneous requests for the same room/time: at most one
may acquire the conflicting allocation. - Requests for adjacent
non-overlapping time ranges behave according to the documented interval
convention. - Concurrent quantity requests cannot allocate more than
stock. - A multi-resource request failing on one item leaves no partial
allocation. - Approval across departments cannot produce an invalid
final state. - Cancellation/rejection releases capacity as intended. -
Serialization/deadlock retry behaviour is bounded and observable. -
Repeated client submission does not create unintended duplicate
reservations if idempotency is implemented.

Create a repeatable concurrency test harness under `tests/concurrency/`.
Use a dedicated test database, not a production database. Record test
parameters and actual outcomes.

Acceptance criteria: - Database invariants remain valid under concurrent
tests. - Tests use real PostgreSQL behaviour. - The transaction design
is documented well enough to defend academically. - No performance claim
is made without measurements.

## Stage 08 --- Teacher availability and syllabus

**Goal:** Implement academic scheduling modules with clear ownership.

### Prompt

Implement teacher availability/office hours and syllabus/course
milestone management according to approved requirements.

Requirements: - Teachers can manage only their own permitted
availability and syllabus records unless an explicit admin permission
says otherwise. - Students can view the approved information needed for
consultations and academic planning. - Validate time ranges, recurrence
rules if supported, and date/timezone handling. - Avoid creating a
complex timetable engine; full university-wide timetable optimisation is
out of scope. - Define whether changes need approval or are immediately
published. - Expose stable API contracts and pagination/filtering where
needed. - Add role, ownership, validation, and date-boundary tests.

Acceptance criteria: - Teacher and student views follow permissions. -
Invalid or conflicting time data is handled according to documented
rules. - API documentation and tests are updated.

## Stage 09 --- College Calendar

**Goal:** Implement the public-facing college events and dates module.

### Prompt

Implement the College Calendar as a separate module from the Resource
Calendar.

It includes college holidays, closures, events, important dates, and
general announcements as approved in requirements.

Permissions: - All intended users can view published college calendar
items. - System Admin and explicitly authorised department users can
create, edit, publish, or archive items. - A user cannot gain editing
privileges simply by selecting a role at registration. -
Draft/unpublished entries must not be exposed to unauthorised users.

Include: - date/time and all-day event handling - college-local timezone
display policy - event status and publication policy - validation and
filtering by date/type - appropriate audit information for changes

Do not place resource reservations into this calendar merely to create a
single combined event feed. The Resource Calendar remains a distinct
module with distinct permissions.

Tests must cover read permissions, edit permissions, unpublished items,
invalid dates, and timezone boundaries.

## Stage 10 --- Resource Calendar API

**Goal:** Provide a permission-safe calendar view for availability and
reservations.

### Prompt

Implement Resource Calendar endpoints separately from College Calendar
endpoints.

Requirements: - Students can view permitted availability and relevant
booking status. - Department users can view and manage information for
resources assigned to their department. - Teachers should follow the
exact access policy documented in the requirements; do not infer
access. - Do not expose another student's unnecessary personal or
booking information. - Support date-range filtering and efficient
querying. - Represent unique-resource bookings and quantity-based
availability correctly. - Distinguish pending approval,
approved/confirmed, cancelled, rejected, and unavailable periods as
defined by the state machine. - Ensure displayed availability does not
bypass transactional checks when a booking is submitted.

Tests must cover role restrictions, cross-department isolation, privacy,
date-range boundaries, and consistency with the reservation engine.

## Stage 11 --- Notification outbox and email

**Goal:** Send reliable emails without making booking correctness depend
on the email provider.

### Prompt

Implement a database-backed notification outbox and background delivery
worker.

Requirements: - Create notification intent within the same database
transaction as the relevant booking/calendar state change. - Send emails
using Nodemailer with provider credentials supplied through environment
variables. - Track attempts, last error (redacted), retry schedule, sent
time, and terminal failure. - Use bounded retries with backoff and avoid
duplicate messages where practical. - A temporary SMTP outage must not
silently undo a valid booking transaction. - Add safe operational
logging and a way to inspect failed notifications in development. - Do
not expose SMTP credentials or raw sensitive content in logs. - Provide
a local test strategy using a mock/test SMTP service or equivalent; do
not claim real email delivery without testing the configured provider.

Tests: - successful delivery - transient failure and retry - permanent
failure - worker restart/reprocessing - duplicate-processing behaviour -
reservation commit while email delivery is unavailable

Acceptance criteria: - The outbox survives process restarts. -
Notification failures are visible and retryable. - Email delivery is
decoupled from the critical reservation transaction.

## Stage 12 --- API contract freeze and integration handoff

**Goal:** Prepare a stable interface for web and mobile work.

### Prompt

Review the implemented backend and create/update `docs/api-contracts.md`
and the OpenAPI specification if used.

For every frontend-required endpoint, document: - method and path -
authentication requirements - roles and object-level permissions -
request body/query parameters - successful response - validation and
error responses - pagination and filtering - date/time formats - token
refresh behaviour - relevant state transitions

Check that the web and mobile clients can use the same backend contract.
Identify inconsistent response formats, missing endpoints, undocumented
assumptions, and unimplemented features.

Do not change business logic simply to make a UI easier without
documenting the decision and updating tests. Create a mock API contract
for design work only if clearly marked as a temporary design aid.

Acceptance criteria: - The contract matches the actual backend
implementation. - Example requests/responses are based on real DTOs. -
Authentication and role behaviour are explicit. - Frontend agents can
work without inventing endpoints.

## Stage 13 --- Stitch design phase

**Goal:** Create a coherent design system while backend work is underway
or nearing completion.

### Prompt for Stitch

Design CampusSpace, a modern college resource and information platform
for Herald College Kathmandu. Produce a coherent, responsive web design
system and a separate mobile experience. This is a design task, not
permission to change business rules.

Primary roles: - Student - Teacher - Department / Resource Manager -
System Admin

Core screens: 1. Sign in and registration. 2. Role-appropriate
dashboard. 3. College Calendar: holidays, closures, events, important
dates, announcements. 4. Resource discovery and availability. 5.
Resource Calendar: room/equipment bookings and availability. 6.
Multi-resource reservation form and review step. 7. Reservation history
and status. 8. Department dashboard for resource catalogue and approval
queue. 9. Teacher availability and office hours. 10. Syllabus/course
milestones. 11. Admin management for users, roles, departments, and
College Calendar content. 12. Notification status or inbox where
required.

Design rules: - College Calendar and Resource Calendar must look and
behave as separate modules. - All reservations require approval; show
pending, approved, rejected, and cancelled states clearly. - Students
must review the full multi-resource request before submission. - Provide
loading, empty, error, permission-denied, and success states. - Make
status understandable without relying on colour alone. - Use accessible
contrast, visible focus states, labels, keyboard navigation on web, and
touch-friendly mobile controls. - Avoid overcrowded dashboards and
decorative charts without a clear purpose. - Do not expose private
details of other users' bookings. - Use realistic but fictional sample
content. - Design responsive desktop, tablet, and mobile web layouts,
plus native-feeling mobile layouts for Expo. - Use a consistent spacing
scale, typography, colour tokens, component states, and icon style. -
Treat any generated React/web code as a reference to adapt, not as proof
that the design is integrated or accessible.

Deliver: - Sitemap/navigation by role - Design tokens and component
guidance - Key screen layouts - Interaction notes for the booking flow -
Responsive behaviour notes - Exportable assets or code where supported -
List of assumptions requiring product-owner approval

Do not invent additional product modules beyond the agreed scope.

## Stage 14 --- Next.js web application

**Goal:** Implement the web UI against the real API and approved Stitch
designs.

### Prompt

Implement the CampusSpace web application in `apps/web` using Next.js
and TypeScript. First inspect the existing API contract, auth
documentation, Stitch outputs, design tokens, and repository structure.

Requirements: - Use the agreed Next.js routing and rendering approach;
document the choice. - Implement role-aware navigation and pages. -
Build accessible reusable components consistent with the design
system. - Integrate the actual API; clearly isolate any temporary mocks
and remove them before acceptance. - Implement secure web authentication
using the agreed cookie/refresh strategy. Do not store long-lived tokens
in localStorage. - Handle expired access tokens, refresh failures,
logout, and unauthenticated navigation safely. - Treat server-side
authorization as authoritative; client-side role checks are only a UX
layer. - Implement College Calendar and Resource Calendar as separate
routes/modules. - Build resource browsing, multi-resource request
review, approval status, reservation history, department approval
workflows, teacher availability, and syllabus views. - Include loading,
empty, error, success, and permission-denied states. - Validate forms
and show actionable errors without leaking internal details. - Ensure
responsive layouts and keyboard accessibility.

Tests: - unit/component tests where useful - Playwright for
registration/login, role-specific access, resource search, booking
request, approval status, and calendar navigation - ensure forbidden UI
paths are also denied by the API

Acceptance criteria: - Main journeys work against the real backend. - No
untracked mock path is mistaken for production integration. - Typecheck,
lint, build, and selected E2E tests run successfully or blockers are
documented.

## Stage 15 --- Expo mobile application

**Goal:** Implement the mobile client against the same API.

### Prompt

Implement `apps/mobile` using Expo, React Native, and TypeScript.
Inspect the API contracts, auth design, Stitch mobile screens, and
shared packages first.

Requirements: - Target Android and iOS; use platform-aware design and
navigation. - Use Expo SecureStore for refresh credentials according to
the approved authentication architecture. - Do not put secrets or
long-lived tokens in plain AsyncStorage. - Use the shared API
contract/client where practical. - Implement sign-in/registration,
dashboard, College Calendar, permitted Resource Calendar, resource
browsing, multi-resource request, reservation status/history, and
role-specific screens required by scope. - Department approvals and
teacher workflows should be included where agreed and feasible. - Handle
offline/network errors clearly; do not claim offline booking support
unless implemented. - Keep manual booking fully functional independently
of voice recognition. - Avoid assuming web browser APIs exist in native
iOS/Android runtime. - Include loading, empty, error, permission-denied,
and success states. - Test on Android emulator/device and iOS
simulator/device when available. Report unavailable platform tests
honestly.

Acceptance criteria: - Core mobile journeys use the actual backend. -
Authentication works with the mobile storage strategy. - Permissions are
enforced by the API. - Typecheck and available Expo checks pass. -
Platform-specific limitations are documented.

## Stage 16 --- Voice-assisted booking (later phase)

**Goal:** Add speech input without allowing speech parsing to decide
whether a booking is valid.

### Prompt

Implement the optional voice-assisted booking feature only after manual
reservation works reliably on web and mobile.

Initial scope: - On supported web browsers, use the Web Speech API for
speech recognition. - Clearly detect unsupported browsers and
recognition errors. - Convert speech into a draft request containing
candidate resource names, date, time, and quantities. - Parse ambiguous
or missing fields into an editable draft, not a final booking. - Require
the user to review and explicitly submit the normal reservation form. -
The backend validates all fields and runs the same booking transaction
and approval flow as manual requests. - Do not let voice input bypass
permissions, availability checks, approvals, quantity limits, or
database constraints. - Keep manual booking available at all times. - Do
not send audio or sensitive information to a third party unless the
provider and privacy implications have been approved. - If a separate AI
parser is later used, isolate it behind a service interface and treat
its output as untrusted input. - Explain that Web Speech API support
differs by browser and platform. Do not promise equivalent speech
functionality in native mobile without a separately evaluated solution.

Tests: - unsupported API - permission denial - recognition failure -
missing/ambiguous fields - user edits parsed fields - invalid
resource/date/quantity - cancellation before submit - backend rejects a
conflict even if the draft looked available

Acceptance criteria: - Speech never directly commits a reservation. -
User confirmation is mandatory. - Manual booking remains functional when
speech fails.

## Stage 17 --- Integration, security, and regression

**Goal:** Verify the full system before deployment.

### Prompt

Run a structured integration and regression review across API,
PostgreSQL, web, mobile, calendar modules, and notifications.

Review: - authentication/session lifecycle - role and object-level
authorization - department isolation - input validation and safe
errors - secrets and environment handling - reservation state
transitions - unique-resource overlap protection - quantity allocation
under concurrency - multi-resource atomicity - notification
outbox/retry - timezone and date-range boundaries -
responsive/accessibility behaviour - web/mobile API consistency -
dependency and configuration risks

Run unit, API integration, concurrency, web E2E, typecheck, lint, and
build checks that are available. Record exact commands and outcomes. Do
not label an unrun test as passed. Prioritise security and
data-integrity bugs over styling polish.

Produce: - `docs/test-plan.md` - `docs/test-results.md` - a list of
known defects and severity - a release-blocker checklist - evidence
needed for the final report

Do not deploy to production or use real user data without explicit
approval.

## Stage 18 --- Deployment preparation

**Goal:** Prepare a repeatable deployment without prematurely choosing a
provider.

### Prompt

Prepare CampusSpace for deployment while the hosting provider is still
undecided.

Tasks: - identify environment variables and secrets needed for each
app - document local/staging/production configuration - verify database
migration and backup/restore procedures appropriate to the chosen
provider - document CORS, cookie, HTTPS, allowed origins, and mobile API
URL configuration - verify that admin provisioning is safe in
production - ensure debug endpoints, seed credentials, and test-only
routes are not exposed - document email SMTP setup and notification
health checks - document database connection limits and connection
pooling constraints for a free-tier host - document logs, basic
monitoring, and operational recovery steps - produce a provider-neutral
deployment guide and later a provider-specific supplement once chosen

Do not claim production readiness merely because the application builds
locally. Clearly distinguish local verification from deployed
verification.

## Stage 19 --- Academic evaluation and report evidence

**Goal:** Produce real evidence for the final-year report.

### Prompt

Use actual implementation and test results to help prepare the
evaluation section for CampusSpace. Do not invent results, citations,
performance numbers, or user-study findings.

Document: - test environment and versions - schema and concurrency
strategy - test cases and expected outcomes - concurrency workload,
number of requests, confirmed/rejected outcomes, latency, retry counts,
and errors where measured - quantity-inventory tests - multi-resource
atomicity tests - notification failure/retry tests - security and
role-boundary tests - usability feedback only if actually collected with
appropriate permission - limitations, failures, trade-offs, and future
work

Separate: 1. planned evaluation, 2. tests actually executed, 3. measured
results, 4. interpretation, 5. limitations.

Use the project's real results. Keep claims proportionate to the
evidence. Maintain a clear link between the academic question,
implementation decisions, test methodology, and conclusions.

------------------------------------------------------------------------

# Part D --- Shared task wrapper for every implementation request

Paste this short wrapper before any stage-specific prompt when using a
different agent:

> You are working on CampusSpace. Read the Master Project Brief and the
> relevant files in `docs/` before acting. The current task is limited
> to: **\[insert task\]**. First report your understanding, plan, files
> likely to change, dependencies, and acceptance criteria. Inspect
> existing code before editing. Do not implement unrelated features or
> change approved architecture silently. If a critical requirement is
> unclear, ask before proceeding. Make small changes, run relevant
> checks, and report the exact commands and actual results. Update
> documentation and `docs/decisions.md` if a decision changes. End with
> a handoff: summary, files changed, tests/results, known issues, and
> next step. Do not claim unrun tests passed.

# Part E --- Code-review prompt

Use this with a second agent after a stage is implemented. Give it the
relevant diff, test output, and requirement files. Ask it to review
rather than rewrite.

> Review this CampusSpace change as an independent senior engineer. Read
> the relevant requirements, architecture, API contracts, and database
> design. Do not edit files yet. Look for:
>
> 1.  functional gaps against acceptance criteria
> 2.  security/authorization flaws, including object-level access
> 3.  race conditions and incorrect transaction boundaries
> 4.  quantity over-allocation and partial multi-resource commits
> 5.  incorrect state transitions
> 6.  token/cookie/refresh-session weaknesses
> 7.  migration or data-integrity risks
> 8.  frontend/backend contract mismatches
> 9.  missing tests or tests that do not prove the claim
> 10. unnecessary complexity or duplicated logic
>
> For each finding, provide severity, file/area, concrete failure
> scenario, why it matters, and a suggested fix. Distinguish confirmed
> defects from questions or speculative risks. Do not praise code
> generically. Do not claim to have run tests unless you did. After the
> review, wait for approval before modifying anything.

# Part F --- End-of-stage handoff template

Ask each agent to fill this in:

``` text
Stage:
Status: Complete / Partial / Blocked

Summary:
- ...

Files created/changed:
- ...

Commands run:
- ...

Actual results:
- Passed:
- Failed:
- Not run:
- Blocked:

Database/API/schema changes:
- ...

Security and concurrency considerations:
- ...

Decisions made:
- ...

Open questions / known issues:
- ...

Next recommended task:
- ...

Manual verification needed from developer:
- ...
```

# Part G --- Four-month execution strategy

The exact calendar date is not yet confirmed. Use relative weeks and
update this schedule after the requirements stage.

  -----------------------------------------------------------------------
  Period                  Main focus              Exit condition
  ----------------------- ----------------------- -----------------------
  Weeks 1--2              Requirements, role      Developer approves
                          matrix, architecture,   scope, schema, and
                          repo, database design   major decisions

  Weeks 3--4              Backend foundation and  Auth and role tests
                          authentication          pass

  Weeks 5--7              Resource catalogue and  Real PostgreSQL
                          reservation engine      concurrency and
                                                  atomicity tests pass

  Weeks 8--9              Resource/College        API contracts and
                          Calendars and           integration tests are
                          teacher/syllabus        stable
                          modules                 

  Weeks 10--11            Notifications and API   Outbox/retry behaviour
                          contract freeze         tested; frontend
                                                  contract approved

  Weeks 8--12,            Stitch design and       Core web journeys work
  overlapping             Next.js implementation  against real API

  Weeks 12--14            Expo mobile             Core mobile journeys
                          implementation and      work against real API
                          integration             

  Week 15                 Voice assistant if core Voice only creates
                          work is stable          editable drafts; manual
                                                  flow remains reliable

  Week 16                 Regression, concurrency Release checklist,
                          evaluation, deployment  results, limitations,
                          preparation, report     and demonstration ready
                          evidence                
  -----------------------------------------------------------------------

This schedule is intentionally aggressive. Work in parallel only where
dependencies are stable: Stitch can design while backend work proceeds;
documentation can be drafted alongside implementation; independent
reviews can happen after a task is complete. Do not parallelise database
schema changes or booking transaction changes across multiple coding
agents.

If time slips, protect these priorities in order: 1. authentication and
authorization 2. correct, atomic reservation handling 3. real PostgreSQL
concurrency and quantity tests 4. usable web core flows 5. usable mobile
core flows 6. teacher/syllabus and calendar completeness 7. notification
resilience 8. voice assistant polish and nonessential visual features

The proposal includes the voice assistant as an extended objective, but
the developer has selected the full proposal as the target scope.
Therefore, it remains planned, with a clear fallback to defer it if core
integrity or the deadline is at risk.

------------------------------------------------------------------------

# Part H --- Before implementation starts: decisions that need explicit approval

The agents must not silently decide these points:

1.  **Pending approval capacity:** Does a pending request hold the
    resource/quantity until approval, or is capacity allocated only
    after all approvals? Recommended: hold capacity with a clear
    expiry/rejection/cancellation release policy.
2.  **Multi-department requests:** Does each department approve its own
    items, and does the entire reservation become approved only after
    all departments approve? Recommended: yes, with one atomic overall
    confirmation policy.
3.  **Teacher Resource Calendar access:** The current decision
    explicitly grants access to Students and Departments. Decide whether
    teachers get no access or a limited availability-only view.
4.  **College Calendar visibility:** "Accessible to everyone" can mean
    all registered users or publicly accessible without login. Decide
    which is intended.
5.  **Registration verification:** Decide whether email verification is
    required for Student/Teacher registration in the first version.
6.  **Department user model:** Decide whether a Department user
    represents one department, and whether a person can be assigned to
    more than one department.
7.  **College timezone and booking boundaries:** Confirm timezone,
    opening hours, minimum/maximum booking duration, booking horizon,
    and whether adjacent intervals count as non-overlapping.
8.  **Notification provider:** Nodemailer is a library, not an
    email-delivery provider. Select a free-tier SMTP provider later and
    verify its sending limits and terms.
9.  **Admin bootstrap:** Agree on the safe one-time provisioning process
    and how the first admin credential is delivered.
10. **Deployment provider:** Defer the choice, but keep environment
    configuration, HTTPS, cookie settings, database limits, and API URLs
    deployment-aware.

**Start with Stage 01.** Do not ask the coding agent to build the whole
application immediately. The first successful outcome is a requirements
and decision baseline that you understand and approve. Then proceed to
the repository and schema.
