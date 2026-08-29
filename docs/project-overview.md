# BACTrack

BAC document tracking and monitoring system for the ICT Department, DepEd Talisay City Division.

This file is a short pointer. All real project rules, decisions, and reference material live in `docs/`. Read `docs/architecture.md` first, then the other files as needed for the task at hand.

## Docs index

- `docs/architecture.md` — stack, layers, and why each choice was made
- `docs/database-schema.md` — every table, column, and relationship
- `docs/roles-and-permissions.md` — the 6 roles and what each can do
- `docs/workflows.md` — document lifecycle, statuses, assignment rules
- `docs/security.md` — auth, rate limiting, audit trail rules
- `docs/open-decisions.md` — things not yet settled; do not silently assume an answer
- `docs/progress.md` — running log of what's built, updated as milestones finish

## Working rules

- Keep changes scoped to the milestone being worked on. Don't build ahead into a later milestone's features.
- The `document_movements` table is append-only. No update or delete on existing rows, from any code path, ever.
- Before adding a new table, column, or package, check `docs/database-schema.md` and `docs/architecture.md` first — if it's not there, it needs a decision, not a silent addition.
- After finishing a milestone, update `docs/progress.md` with what was built and what's next.
