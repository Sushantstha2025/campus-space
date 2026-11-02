# CampusSpace — Decision Register

> Canonical record of project decisions. **A recommendation in the master brief is not an approval.** Only the developer can approve a decision.
>
> Last updated: 2026-11-02
>
> Status values: **Proposed** / **Pending Approval** / **Approved** / **Rejected**

## Register conventions

- Every consequential decision gets an ID and is never silently approved.
- When a decision changes, update it here first, then assess the impact on the API, database schema, tests, web app, mobile app and project report.
- "Developer approval reference" records how/when approval was given. Until then it stays `None`.
- Rejected and superseded decisions are retained; do not delete history.

## Summary

| ID | Topic | Status | Date |
|---|---|---|---|
| D-001 | Pending-request capacity holds | Pending Approval | 2026-11-02 |
| D-002 | Multi-department approval aggregation | Pending Approval | 2026-11-02 |
| D-003 | Teacher Resource Calendar access | Pending Approval | 2026-11-02 |
| D-004 | College Calendar visibility | Pending Approval | 2026-11-02 |
| D-005 | Registration email verification | Pending Approval | 2026-11-02 |
| D-006 | Department user model | Pending Approval | 2026-11-02 |
| D-007 | College timezone and booking boundaries | Pending Approval | 2026-11-02 |
| D-008 | Notification / SMTP provider | Pending Approval | 2026-11-02 |
| D-009 | Admin bootstrap and first credential | Pending Approval | 2026-11-02 |
| D-010 | Deployment provider | Pending Approval | 2026-11-02 |

---

## D-001 — Pending-request capacity holds

- **Status:** Pending Approval
- **Date:** 2026-11-02
- **Description:** Decide whether a reservation that is awaiting department approval immediately holds the required resource/quantity capacity, or whether capacity is allocated only after all approvals succeed.
- **Options:** (a) Hold capacity transactionally while pending, with explicit expiry/cancellation/rejection release rules; (b) do not hold capacity, allocate only at final approval.
- **Brief recommendation:** Option (a) — hold capacity, with a clear expiry/rejection/cancellation release policy.
- **Reasoning:** Holding capacity prevents more requests being approved than the resource can satisfy; not holding it reduces stale holds but creates a risk of approval-time conflicts and poor user experience.
- **Impacted modules:** reservations, approvals, availability, Resource Calendar, database schema/transactions, concurrency tests, notification outbox.
- **Developer approval reference:** None — awaiting developer decision.

## D-002 — Multi-department approval aggregation

- **Status:** Pending Approval
- **Date:** 2026-11-02
- **Description:** Decide how a multi-resource request spanning multiple departments is approved, and when the overall reservation becomes approved.
- **Options:** (a) Each department approves its own resource lines; the whole reservation becomes approved only when every required department has approved, with one atomic overall confirmation; (b) some other aggregation/partial-approval model.
- **Brief recommendation:** Option (a).
- **Reasoning:** Matches departmental ownership, avoids one department approving another's resources, and preserves atomic confirmation.
- **Impacted modules:** reservations, approvals, authorization, database transactions, Resource Calendar, notifications, tests.
- **Developer approval reference:** None — awaiting developer decision.

## D-003 — Teacher Resource Calendar access

- **Status:** Pending Approval
- **Date:** 2026-11-02
- **Description:** The current requirement grants Resource Calendar access to Students and authorised Department users. Decide whether Teachers receive no access or a limited availability-only view.
- **Options:** (a) No Resource Calendar access; (b) limited availability-only view; (c) another explicitly approved scope.
- **Brief recommendation:** None; the brief requires the developer to decide because it is ambiguous.
- **Reasoning:** Teachers may need availability for consultation scheduling, but granting booking details could expose private information beyond their role's need.
- **Impacted modules:** authorization, Resource Calendar API/UI, privacy rules, tests.
- **Developer approval reference:** None — awaiting developer decision.

## D-004 — College Calendar visibility

- **Status:** Pending Approval
- **Date:** 2026-11-02
- **Description:** Decide whether "accessible to everyone" means all registered/authenticated users or public access without login.
- **Options:** (a) All authenticated users only; (b) public read access without login; (c) another explicitly approved policy.
- **Brief recommendation:** None; the brief notes this may be public if the product owner confirms.
- **Reasoning:** Public visibility affects authentication middleware, caching, rate limiting, privacy and deployment/origin configuration.
- **Impacted modules:** College Calendar, authentication/authorization, API, web/mobile navigation, deployment configuration.
- **Developer approval reference:** None — awaiting developer decision.

## D-005 — Registration email verification

- **Status:** Pending Approval
- **Date:** 2026-11-02
- **Description:** Decide whether email verification is required for Student/Teacher registration in the first version.
- **Options:** (a) Require verification before account use; (b) defer verification; (c) another approved onboarding policy.
- **Brief recommendation:** None; the brief asks for an explicit decision and warns not to invent a complete production identity system without discussing effort.
- **Reasoning:** Verification improves account integrity but adds email-delivery dependency, token flows and onboarding complexity within a four-month solo timeline.
- **Impacted modules:** authentication, registration, notification outbox/email, user schema, web/mobile onboarding, tests.
- **Developer approval reference:** None — awaiting developer decision.

## D-006 — Department user model

- **Status:** Pending Approval
- **Date:** 2026-11-02
- **Description:** Decide whether a Department/Resource Manager account represents exactly one department, and whether one person may be assigned to more than one department.
- **Options:** (a) One department per Department user; (b) multiple departments per user with explicit assignments; (c) another approved model.
- **Brief recommendation:** None; the brief asks for an explicit decision.
- **Reasoning:** The choice changes the schema, authorization checks, approval routing and department-isolation tests.
- **Impacted modules:** users/roles, departments, authorization, reservations/approvals, database schema, tests.
- **Developer approval reference:** None — awaiting developer decision.

## D-007 — College timezone and booking boundaries

- **Status:** Pending Approval
- **Date:** 2026-11-02
- **Description:** Confirm the college-local timezone; opening hours; minimum and maximum booking duration; booking horizon; and whether adjacent intervals (for example 09:00–10:00 and 10:00–11:00) count as non-overlapping.
- **Options:** To be specified by the developer. Storage remains UTC; display and business rules use the confirmed college-local timezone.
- **Brief recommendation:** Confirm the intended timezone before implementing time-sensitive rules. Adjacent-interval convention must be documented.
- **Reasoning:** These rules determine range boundaries, validation, availability queries and concurrency-test expectations; guessing them would invalidate reservation behaviour.
- **Impacted modules:** reservations, availability, Resource Calendar, College Calendar, teacher availability, validation, database constraints, tests, report.
- **Developer approval reference:** None — awaiting developer decision.

## D-008 — Notification / SMTP provider

- **Status:** Pending Approval
- **Date:** 2026-11-02
- **Description:** Select the email-delivery provider and verify its free-tier sending limits and terms. Nodemailer is a library, not a provider.
- **Options:** To be selected later from free-tier SMTP providers; credentials supplied through environment variables.
- **Brief recommendation:** Choose later; keep configuration environment-driven.
- **Reasoning:** Provider limits/terms affect retry policy, deliverability testing and operational readiness.
- **Impacted modules:** notification outbox/worker, configuration, deployment, tests, report.
- **Developer approval reference:** None — awaiting developer decision.

## D-009 — Admin bootstrap and first credential

- **Status:** Pending Approval
- **Date:** 2026-11-02
- **Description:** Agree the safe one-time provisioning process for the first System Admin and how the initial credential is delivered.
- **Options:** (a) Explicit seed/setup command using environment-supplied credentials; (b) another controlled setup mechanism. Never source-code literals.
- **Brief recommendation:** Use a deliberate seed/initial setup procedure with environment-supplied credentials or a controlled setup mechanism.
- **Reasoning:** The bootstrap path is a privilege-escalation and secrets-management risk; it must be safe in both development and deployment.
- **Impacted modules:** authentication, authorization, seeding, deployment, environment configuration, security tests.
- **Developer approval reference:** None — awaiting developer decision.

## D-010 — Deployment provider

- **Status:** Pending Approval
- **Date:** 2026-11-02
- **Description:** Select the hosting/deployment provider and database hosting approach.
- **Options:** Deferred; keep environment configuration, HTTPS, cookie settings, connection limits and API URLs deployment-aware.
- **Brief recommendation:** Defer the choice while remaining deployment-aware.
- **Reasoning:** Provider constraints affect cookies/CORS, TLS, connection pooling, migrations, backups and email configuration.
- **Impacted modules:** deployment configuration, API/web/mobile environment handling, database operations, security, report.
- **Developer approval reference:** None — awaiting developer decision.

---

## Established baseline (not pending decisions)

The master project brief establishes the following and they must not be silently changed: project purpose and module scope; one Git monorepo; the Node/Express/TypeScript/PostgreSQL/Prisma backend; Next.js web and Expo mobile clients; the four roles; approval-based reservations; PostgreSQL as the final authority for reservation integrity; atomic multi-resource allocation; outbox-based notifications; and the security/testing baseline.

Changing any of these requires a new decision entry and explicit developer approval.
