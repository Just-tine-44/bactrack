# Architecture

## What this system does

BACTrack does not change how BAC (Bids and Awards Committee) documents get approved. Documents still move on paper and get physically signed. BACTrack adds a digital layer on top: every time a document changes hands, someone scans a QR code, and the system records who has it, what status it's in, and how long it's been sitting there. This gives management visibility into where documents are getting stuck.

## Scope for this build

One pilot office: Talisay City Schools Division Office. BAC members come from different departments within that office (Procurement, Legal, Finance, etc.), so the system does track which department currently holds a document, not just which person. It does not need to support multiple Schools Division Offices at once.

## Stack

| Layer | Choice | Why |
|---|---|---|
| Backend | Laravel | Team's existing stack, mature ecosystem, built-in queue/scheduler/auth tools |
| Frontend | Vue 3 + Inertia.js | No separate REST API layer needed — Inertia sends data straight from Laravel controllers to Vue pages. Less code to write and secure. |
| Styling | Tailwind CSS | Team's existing stack |
| Database | PostgreSQL | Team's existing stack, matches TALNAS |
| QR codes | `endroid/qr-code` package | Generates a QR image server-side; QR encodes a link, not raw document data |
| Notifications | Laravel's built-in mail + queue | No separate email service needed for pilot scale |
| Hosting | Public web hosting, HTTPS required | System is reachable from outside the DepEd network; also HTTPS is required for QR scanning to work on phones at all |

## Why no separate REST API

Since only the web app is planned for now (no mobile app), Inertia removes the need to build and maintain a separate API layer. If a mobile app or outside integration becomes a real requirement later, that's a new decision to make then — not something to build speculatively now.

## Why no in-app camera scanner

Each document's QR code encodes a private link (e.g. `/q/{public_id}`), not document data. When scanned with the phone's normal camera app, it opens that link in the browser, which then requires login and checks whether the logged-in user is allowed to act on that document. This means we don't need to build or maintain a custom camera-scanning screen — any phone's built-in camera already does the job.

## Why no file attachments / e-signatures

Documents remain physically signed on paper. BACTrack tracks custody and status only. This avoids needing file storage, virus scanning, e-signature legal validity questions, and the security surface that comes with file uploads — none of which the pilot needs.

## Layers, top to bottom

1. **Browser (phone or PC)** — login, QR scan (via phone's own camera), status update forms
2. **Laravel + Inertia + Vue app** — authentication, role checks, business rules (who can act on what), page rendering
3. **PostgreSQL database** — users, documents, movements, notifications
4. **Background queue** — sends emails, runs the daily aging/overdue check, without blocking normal page loads

## Deliberately not included in this MVP

- Configurable workflow templates (the step sequence per document type) — the sequence is handled by whoever currently holds the document choosing the next recipient, not a predefined chain
- Business-hours/holiday-aware time calculations — aging is calculated on raw elapsed time for now (see `docs/open-decisions.md`)
- A separate permissions table — roles are a fixed, small list, so a single `role` column on `users` is enough (see `docs/roles-and-permissions.md`)
- Reports/exports beyond the dashboard
