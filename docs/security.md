# Security

## Authentication

- Laravel Breeze for login, password reset, session handling — don't hand-roll this
- Passwords hashed with bcrypt (Laravel's default)
- Session-based auth, not API tokens (no separate API exists — see `docs/architecture.md`)

## Authorization

- See `docs/roles-and-permissions.md` for the role model and the two-layer enforcement approach

## HTTPS

Required everywhere, not optional. Phone browsers only allow camera access over HTTPS, and this is a public-facing system handling government process data.

## Rate limiting

Apply Laravel's `throttle` middleware to:
- the login route
- the QR scan / document-action routes

This stops both password-guessing and someone spamming scan actions.

## Audit trail integrity

`document_movements` is append-only. No code path — including admin tools — should ever update or delete a row in this table. This is what makes the history trustworthy for both the government process and the research paper's data.

## Mass assignment protection

Use `$fillable` (explicit allow-list) on every model, not `$guarded = []`. Being explicit about which fields can be set from user input is a small amount of extra typing that prevents a real class of bugs.

## Environment and secrets

- `.env` is never committed to git
- `APP_KEY` and any credentials are rotated if they were ever exposed
- The institutional email account used for sending notifications uses an app password, not the real account password

## Backups

Automated daily database backup, stored somewhere separate from the live server. Minimum requirement before this goes anywhere near production data.

## Report visibility

The dashboard/aging reports show which specific person is holding up a document. This is useful for management but is also information about individual staff performance. Keep this visible only to `management`, `ict_admin`, and `researcher` roles — not to `bac_member` or `procurement` looking at documents that aren't theirs.

## Frontend assets

Bundle CSS/JS locally via Vite, don't load from an external CDN. Government networks sometimes block or throttle outside websites, and a CDN dependency means the whole interface can break for reasons outside your control.
