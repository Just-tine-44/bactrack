# Database schema

All tables use PostgreSQL. Timestamps mean the standard `created_at`/`updated_at` pair unless stated otherwise.

## `units`

Represents an office or department within the Talisay SDO (e.g. Procurement, Legal, Finance). BAC members belong to different units, and documents move between units, not just between individuals.

| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| parent_id | bigint FK → units, nullable | Self-reference, for sub-units if ever needed |
| code | string, unique | Short code, e.g. `PROC`, `LEGAL` |
| name | string | Full name |
| is_active | boolean, default true | |
| timestamps | | |

## `users`

| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| unit_id | bigint FK → units | Which department this person belongs to |
| name | string | |
| email | string, unique | |
| password | string, hashed | |
| role | string | One of: `secretariat`, `bac_member`, `procurement`, `management`, `ict_admin`, `researcher` — see `docs/roles-and-permissions.md` |
| is_active | boolean, default true | Inactive users cannot log in |
| last_login_at | timestamp, nullable | |
| timestamps | | |

## `document_types`

Admin-manageable lookup table (not a fixed code list), so ICT Admin can add new types without a code change.

| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| code | string, unique | e.g. `BAC-RES` |
| name | string | e.g. "BAC Resolution" |
| target_minutes | integer, nullable | Expected processing time for this type, used for aging calculations |
| is_active | boolean, default true | Deactivate instead of delete, so old documents keep a valid reference |
| timestamps | | |

## `documents`

One row per physical document.

| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| public_id | ULID, unique | Used in the QR code link — not the sequential id, so URLs aren't guessable |
| tracking_number | string, unique | Human-readable number staff refer to, e.g. `BAC-2026-000123` |
| document_type_id | bigint FK → document_types | |
| title | string | |
| current_status | string | One of the workflow statuses — see `docs/workflows.md`. Duplicated here from the latest `document_movements` row for fast list/dashboard queries. |
| current_assignee_id | bigint FK → users, nullable | Who is expected to act next. Enforced on every status-changing action. |
| current_unit_id | bigint FK → units, nullable | Which department currently holds it |
| registered_by | bigint FK → users | Who registered the document originally |
| priority | string, default `normal` | |
| remarks | text, nullable | |
| registered_at | timestamp | |
| completed_at | timestamp, nullable | |
| cancelled_at | timestamp, nullable | |
| timestamps | | |

## `document_movements`

Append-only. One row per QR scan / status change. This is the full audit trail — turnaround time, aging, and history are all calculated from this table's timestamps, never stored separately.

| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| document_id | bigint FK → documents, cascade | |
| acted_by | bigint FK → users | Who performed this action |
| from_status | string, nullable | Null for the very first row |
| to_status | string | |
| from_unit_id | bigint FK → units, nullable | |
| to_unit_id | bigint FK → units, nullable | |
| assigned_to | bigint FK → users, nullable | Who this hands off to next — copied onto `documents.current_assignee_id` when the row is created |
| remarks | text, nullable | e.g. reason for returning a document |
| ip_address | string, nullable | Captured for audit credibility |
| user_agent | string, nullable | Captured for audit credibility |
| occurred_at | timestamp | |

No `updated_at`/soft delete on this table on purpose — nothing should ever modify a row after it's written.

## `notifications`

| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| document_id | bigint FK → documents, cascade | |
| recipient_id | bigint FK → users | |
| type | string | `reminder`, `overdue`, `escalation` |
| channel | string | `email` for now |
| sent_at | timestamp, nullable | Null until the queue job actually sends it |
| timestamps | | |

## Design notes

- **No stored duration/turnaround fields anywhere.** Everything time-related (waiting time, processing time, whether something is overdue) gets calculated from `document_movements.occurred_at` timestamps at query time. Storing calculated numbers risks them drifting out of sync with the source data.
- **`documents.current_status` and `current_assignee_id` are intentional duplication** (denormalization) of what the latest `document_movements` row says, kept in sync every time a movement is recorded. This makes document lists and the dashboard fast, since they don't need to search history on every page load.
- **Concurrency:** use database row locking (`lockForUpdate()`) inside a transaction when writing a movement, rather than a separate optimistic-lock version column. Simpler, and does the same job.
- **This table structure is a deliberate simplification** compared to a 3-table split (routes / status history / acknowledgements) seen in an earlier prototype of this system. One append-only table with `ip_address`/`user_agent` captured inline covers the same audit needs with less to maintain, given this is a single-office pilot.
