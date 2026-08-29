# Workflows

## Document statuses

In order, for the normal path:

```
registered → released → received → pending_action → under_review → signed → returned → completed
```

`cancelled` is a terminal status reachable from most in-progress states (e.g. a bid gets cancelled).

- **registered** — Secretariat has entered the document into the system, not yet physically handed to anyone
- **released** — Secretariat has handed it to the first recipient, QR sticker attached
- **received** — recipient scanned the QR to confirm they now physically have it
- **pending_action** — received, but work hasn't started yet
- **under_review** — recipient is actively reviewing/working on it
- **signed** — recipient has physically signed and is ready to hand it onward
- **returned** — handed back to Secretariat (may repeat if there are multiple signatories in sequence)
- **completed** — Secretariat closes the document, process finished

## Assignment rule

Every document has one `current_assignee_id` (a specific person) and one `current_unit_id` (their department) at any given time. Only that person can perform the next status-changing action. This is checked every time, not just at release.

When a movement is recorded, the person recording it chooses who it's handed to next (`assigned_to` on the `document_movements` row). This gets copied onto `documents.current_assignee_id`. There's no predefined, configurable chain of steps for the MVP — the current holder manually chooses the next recipient each time, matching how it works on paper today.

## What happens on a QR scan

1. Recipient scans the QR code with their phone's normal camera
2. It opens the document's link, which requires login
3. If the logged-in user matches `current_assignee_id`, they can act (mark received, start review, sign, return, etc.)
4. If not, they see a clear message that this document is assigned to someone else
5. The action is written as a new row in `document_movements`, inside a transaction, and `documents.current_status` / `current_assignee_id` are updated in the same transaction

## Aging and notifications

Each `document_types` row has a `target_minutes` value — the expected time to process that type. A document's "aging class" (normal / approaching / overdue) is calculated on the fly: current time minus the timestamp of its latest `document_movements` row, compared against the target. This is not stored — see `docs/database-schema.md`.

A scheduled daily job checks all in-progress documents against their target and queues an email (via `notifications`) to the current assignee and, for overdue ones, to Management, when a threshold is crossed. See `docs/open-decisions.md` for what the actual thresholds should be — not yet settled.
