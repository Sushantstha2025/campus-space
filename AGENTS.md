# CampusSpace — Permanent Agent Instructions

CampusSpace is a final-year software engineering project for Herald College Kathmandu: a modular college management and concurrency-safe resource reservation system. It centralises selected college information and provides trustworthy booking of rooms, equipment and quantity-based inventory.

## Canonical specification

`docs/master-project-brief.md` is the canonical project specification. Read it before acting whenever requirements, scope, architecture, data integrity, security or evaluation behaviour needs clarification. Do not edit it without explicit developer instruction.

Do not duplicate or "improve" the brief in these instructions. If this file and the brief conflict, the brief wins and the conflict must be raised with the developer.

## Mandatory documentation-reading procedure

At the start of every session, read:

1. This file.
2. The nearest relevant nested `AGENTS.md` (for backend work: `apps/api/AGENTS.md`).
3. `docs/agent-context/CURRENT_STATE.md`.
4. `docs/agent-context/SESSION_HANDOFF.md`.
5. `docs/agent-context/PROJECT_CONTEXT.md`.
6. `docs/master-project-brief.md` when requirements need clarification.
7. `docs/decisions.md` before making or assuming any architectural, security or data-integrity decision.

Then inspect only the source files relevant to the assigned task. Read existing documentation and code before changing them.

## Inspect before modifying

- Never overwrite, regenerate or delete a file without reading it first.
- Prefer targeted edits. Do not reformat or restructure unrelated code.
- Treat unfamiliar or partially finished work as the developer's work; investigate it before replacing it.
- Check for an existing implementation before creating a new file.
- If a Graphify knowledge graph exists (`graphify-out/graph.json`), use it to locate symbols, dependencies and relationships. Graphify is a navigation aid, not authority: verify important findings in the actual source, and never assume a stored graph matches current code.

## Small-task implementation policy

- Implement only the assigned task or stage. Keep changes small and reviewable.
- Do not add unrelated features, refactors, dependencies, tools or "improvements".
- Do not begin a later stage until the developer has reviewed and accepted the current one.
- Before coding, state your understanding, a short plan, files expected to change, dependencies and acceptance criteria.
- If a requirement is ambiguous, ask. If a safe default is obvious, propose it and record it in `docs/decisions.md`. Do not guess on security or data-integrity matters.

## Architecture and change control

- The approved baseline is defined in the master brief and `docs/decisions.md`. Do not introduce technologies, modules, services or patterns that are not approved there.
- Do not change the database schema, reservation state machine, transaction strategy, authentication flow or API contract without developer approval.
- Record any approved change in `docs/decisions.md` before implementing it, then assess its impact on the API, schema, tests, web app, mobile app and report.
- Do not silently approve a proposed decision.

## Security and data-integrity priorities

- PostgreSQL, not the client or an application-level pre-check, is the final authority for reservation integrity.
- Every protected operation requires backend authentication and authorization, including object-level ownership and department-assignment checks. Hiding UI controls is not authorization.
- Multi-resource reservations must be atomic; no partial allocation may survive a failure.
- Cancelled, rejected and expired reservations must not consume capacity.
- Keep notification delivery outside the critical booking transaction via the outbox pattern.
- Never log passwords, tokens, cookies, secrets or unnecessary personal data.
- Never commit `.env` files, credentials, keys or real user data.
- Treat the master brief's concurrency and role-boundary requirements as non-negotiable unless the developer explicitly amends them.

## Testing and truthful reporting

- Run the relevant format, lint, typecheck, migration and test commands that exist in the project.
- Report the exact commands and their actual results.
- Never claim a test passed unless it was run and passed. Distinguish "passed", "failed", "not run" and "blocked".
- Do not use mocks as evidence that real API/database integration works.
- Do not claim performance, concurrency or security results without actual measurements or evidence.
- Fix the cause of failures; do not disable checks or weaken tests to make a build pass.

## Git and secret-management rules

- One root Git repository; no nested repositories.
- Do not commit generated secrets, local databases, `node_modules`, build output or `.env` files.
- Keep `.env.example` files free of real values.
- Do not rewrite published history or force-push shared branches without explicit approval.
- Do not commit changes the developer has not asked for.
- Inspect `git status` and the diff before committing, and report what was staged.

## Session-start procedure

1. Read the files in the mandatory documentation-reading procedure above.
2. Read `CURRENT_STATE.md` to learn the phase, accepted milestone and immediate next task.
3. Read `SESSION_HANDOFF.md` for the previous session's results, known issues and required manual verification.
4. Confirm the assigned task and its acceptance criteria before editing.
5. If the task conflicts with the recorded state or decisions, stop and ask.

## Session-end procedure

Update only what actually changed:

- `docs/agent-context/CURRENT_STATE.md` — phase, progress, completed/pending/blocked items, next task.
- `docs/agent-context/SESSION_HANDOFF.md` — this session's record.
- `docs/agent-context/PROJECT_CONTEXT.md` — only when established architectural knowledge changes.
- `docs/decisions.md` — only when a decision is proposed, approved or rejected.

Do not create no-op documentation updates just to fill files. Do not mark work complete or accepted until it has actually been verified and, where required, developer-accepted.

## Mandatory handoff requirements

Every session ends with a handoff containing:

- Stage/task and status: Complete / Partial / Blocked.
- Summary of work.
- Files created or modified.
- Commands executed and their actual results.
- Tests/results: passed, failed, not run, blocked.
- Database/API/schema changes (or "none").
- Security and concurrency considerations.
- Decisions made and their references.
- Unresolved questions and known issues.
- The next exact task.
- Manual verification required from the developer.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

When the user types `/graphify`, use the installed graphify skill or instructions before doing anything else.

Rules:
- When graphify-out/graph.json exists, run `graphify query "<question>"` before your first grep, search or source-file read for a codebase question, and put this rule in every subagent prompt that explores code. Read raw files only for lines the graph or an earlier read already located. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- Dirty graphify-out/ files are expected after hooks or incremental updates; dirty graph files are not a reason to skip graphify. Only skip graphify if the task is about stale or incorrect graph output, or the user explicitly says not to use it.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).
- A committed graphify-out/graph.json is rewritten whole on every build, so a large diff to it in a pull request is expected. Do not flag it as churn or suggest skipping the graph rebuild or its hooks.
