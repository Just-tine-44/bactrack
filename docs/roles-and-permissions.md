# Roles and permissions

Six fixed roles, stored as a single `role` string column on `users` (no separate permissions table — see `docs/architecture.md` for why).

| Role | What they can do |
|---|---|
| `secretariat` | Register new documents, release them, generate the tracking QR, view all documents, close/complete documents |
| `bac_member` | Receive, review, sign, return documents assigned to them. Cannot see documents assigned to other members unless also given `management`-level visibility. |
| `procurement` | View documents relevant to procurement, same acting rights as `bac_member` when assigned |
| `management` | Read-only. Views dashboard, aging report, bottleneck report across all documents. Cannot edit or act on documents. |
| `ict_admin` | Full access. Manages user accounts, document types, units. Can view everything. Cannot bypass the "only the assigned person can act" rule on a live document — this stays enforced even for admins, to keep the audit trail meaningful. |
| `researcher` | Read-only. Pulls baseline vs pilot turnaround data for the policy research paper. Same visibility as `management`. |

## Enforcement approach

Use Laravel Policies (one method per action: view, act-on, complete, etc.) that check the user's `role` and, where relevant, whether they are the document's `current_assignee_id`. This is checked at two levels:

1. **Controller/Policy layer** — blocks the request before anything happens
2. **Inside the action itself** — re-checks the same rule right before writing to the database, inside the same transaction, as a second layer of protection

Both layers matter: the second one protects against a request that got past the first check due to a bug elsewhere, or a race condition where two people act on the same document at nearly the same time.

## Why not a permissions table

Roles are few, fixed, and not expected to change during the pilot. A `role_user` / `permission_role` many-to-many setup is worth the extra complexity when permissions vary per user or change often — neither is true here. If DepEd adopts this division-wide later and roles get more complex, moving from a `role` column to a proper permissions table is a normal, well-understood upgrade path.
