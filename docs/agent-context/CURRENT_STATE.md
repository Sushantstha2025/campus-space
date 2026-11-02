# CampusSpace — Current State

> Progress tracker for agents. Update at the end of each session. Do not mark work complete or accepted until it has actually been verified and, where required, approved by the developer.
>
> Last updated: 2026-11-02

## Status summary

| Area | Status |
|---|---|
| Project phase | Initial agent environment setup |
| Current stage | Stage 00 — agent instructions and context documentation only |
| Application implementation | Not started |
| Requirements baseline | Pending |
| Database design | Pending |
| Backend implementation | Not started |
| Web implementation | Not started |
| Mobile implementation | Not started |
| Last accepted milestone | None yet. The planning baseline is the supplied master project brief (24 October 2026). |

## Completed tasks

No task has been accepted by the developer yet.

## Delivered this session (awaiting developer acceptance)

- Root `AGENTS.md` extended with permanent CampusSpace agent instructions while preserving the existing graphify block.
- Backend-specific instructions created at `apps/api/AGENTS.md`.
- Context documents created under `docs/agent-context/`.
- Decision register created at `docs/decisions.md` with the ten unresolved decisions from the master brief marked **Pending Approval**.

These are delivered but not accepted. Do not treat them as an approved baseline until the developer reviews them.

## Pending tasks

1. Developer review and acceptance of this agent-environment setup.
2. Stage 01 — Requirements and scope baseline: produce `docs/requirements.md`, `docs/architecture.md` (high-level draft), `docs/roadmap.md`, `docs/roles-and-permissions.md`, and update `docs/decisions.md`.
3. Developer resolution of decisions D-001 to D-010.
4. Stage 02 — Architecture and repository foundation (requires accepted requirements and decisions).
5. Stage 03 — Data model and PostgreSQL design.
6. Stages 04–19 as defined in the master brief.

## Blocked tasks

- Any application implementation, database design or repository foundation work is blocked until the requirements baseline and relevant decisions are approved.
- Stage 01 must not begin automatically; it requires explicit developer instruction after this setup is accepted.

## Outstanding developer decisions

All remain **Pending Approval**; none are approved by default.

| ID | Topic |
|---|---|
| D-001 | Pending-request capacity holds |
| D-002 | Multi-department approval aggregation |
| D-003 | Teacher Resource Calendar access |
| D-004 | College Calendar visibility (registered users vs. public) |
| D-005 | Registration email verification |
| D-006 | Department user model (one or multiple departments) |
| D-007 | College timezone and booking boundaries |
| D-008 | Notification/SMTP provider |
| D-009 | Admin bootstrap and first-credential delivery |
| D-010 | Deployment provider |

See `docs/decisions.md` for reasoning, impact and recommended defaults.

## Immediate next task

The developer should manually review the files created/modified in this session. After acceptance, the next exact task is **Stage 01 — Requirements and scope baseline**, using the prompt in `docs/master-project-brief.md`. Do not start it until the developer explicitly instructs it.

## Repository state notes

- `docs/master-project-brief.md` is unchanged (SHA-256 `79AF1996924F4D8720C0344849A023208669710C69E60398E69DDF4676A5F0AD`).
- No `package.json`, npm workspace, dependency, database schema, migration, Prisma model or application source exists yet.
- `apps/api/` currently contains only `AGENTS.md`.
- No Graphify graph exists (`graphify-out/graph.json` absent), so graph-based navigation is unavailable in this session.
