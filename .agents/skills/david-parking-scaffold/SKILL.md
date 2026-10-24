---
name: david-parking-scaffold
description: Scaffold a new Cloudflare based parking payment and slip record system from this project's architecture, including D1, private R2, admin access, and a complete environment setup checklist. Use when starting a similar project; do not use for unrelated apps.
metadata:
  short-description: Scaffold a parking records project
---

# David Parking project scaffold

Use this skill to start a new parking-rental payment and slip-record project based on this repository. Treat the current repository as an architectural reference, not as a source of production identifiers, credentials, customer records, or deployment state.

## Preserve these product and security decisions

- Use a plate-centric model: a license plate is the vehicle identity; parking slot and rent can change over time. Preserve a dated/effective history of slot, tenant, fee, and status so old payments keep their original snapshot.
- Treat Cloudflare D1 as the authoritative record store. Store money as integer satang, use bound SQL parameters, and keep audit history for administrative changes.
- Store new slip files in a **private R2 bucket**. Keep public bucket access and `r2.dev` disabled. Serve slip files only through an admin-authorized Worker route. Retain a legacy Drive adapter only if migrating or viewing old Drive-backed records.
- Keep Cloudflare Access as the admin sign-in/perimeter gate **and** validate its signed JWT in the Worker. Check signature, issuer, audience, expiry, and the normalized `ADMIN_EMAILS` allowlist. Access policy and Worker allowlist must contain the same authorized administrators. The Worker allowlist is authorization, not a login mechanism.
- Keep payer submission public but protected by server-verified Turnstile, input validation, rate limits, file type/size checks, and idempotency. Never expose tenant contact data, payment history, or slip objects through public endpoints.
- A submitted slip is provisional evidence, not proof of bank settlement. Support manual approval/rejection and manual allocation of an amount that differs from rent (fees, penalties, or partial/extra payment). Duplicate submissions should be retained and flagged for review rather than silently discarded.
- Generate a receipt/reference with a QR code that opens a limited status/check page; provide a printable/downloadable PDF receipt. Do not make the QR reveal private slip files or personal information.
- Google Sheets, LINE, and email are optional integrations. Start disabled, keep D1 authoritative, and avoid sending unnecessary personal data to integrations. Sheets is a one-way mirror; edits in the Sheet do not update D1.
- Use Bangkok time for user-facing dates and `YYYY-MM` rental periods. Store event timestamps in UTC.

## Scaffold workflow

1. **Inspect and isolate.** Review the current branch, local instructions, requirements, migrations, Worker routes, web UI, and config examples. Never read or copy live `.dev.vars`, secret files, production database contents, license plates, current account IDs, or deployed resource IDs into a new project. Start from a clean new repository or explicitly requested target directory; keep the original project untouched.
2. **Confirm project-specific decisions.** Record the new project name/hostname, admin identities, initial vehicle import format, rent/slot rules, expected traffic, retention/backup needs, and which optional integrations are wanted. Use private R2 by default. Ask only about decisions that cannot be inferred and materially affect schema, access, or storage.
3. **Adapt identity and data.** Rename Worker, D1 database, R2 bucket, static asset paths, app title, email/LINE labels, hostname, and receipt branding. Replace the current parking registry with owner-provided data only. Preserve history and never infer that repeated rows are accidental duplicates without review.
4. **Build the backend and UI.** Create additive D1 migrations; implement vehicle/tenant management, payment intake, slip storage, manual allocation/review, reports, receipts, and admin APIs. Keep protected admin authorization at the Worker boundary. Provide CSV import/export with preview and validation. Use pagination and sortable tables; choose a default page size appropriate to the expected data volume.
5. **Configure external services.** Create new D1/R2 resources and bindings, Access application/policies, Turnstile widget, and optional Apps Script/LINE/email integrations. Use the checklist below. Generate fresh credentials per project and store them in the provider secret manager; do not reuse values from this repository.
6. **Verify before go-live.** Apply migrations to a clean local database, run typecheck/build and the project's tests, verify public and protected routes, test private slip upload/read authorization, CSV import/export, allocation/duplicate rules, receipt QR/PDF, and optional integrations in disabled or test mode. Confirm backups and production hostname configuration. Deploy only when deployment is requested; never import production records or change live resources without explicit authorization.

## Configuration checklist

Use the exact binding/variable names expected by the implementation. The names below match the reference code; omit optional entries that the new project does not implement. Set values in Cloudflare Worker settings or Wrangler's local-only `.dev.vars`, not in frontend source or committed files.

### Cloudflare Worker variables

| Name | Required when | Purpose / setup |
| --- | --- | --- |
| `ENVIRONMENT` | Always | `development` locally; `production` in the deployed Worker. Local token auth must never be enabled on a public host. |
| `APP_ORIGIN` | Production | Exact public origin, including `https://`; used for origin checks and app links. |
| `SLIP_STORAGE` | Always | Use `r2` for new slips. Select `drive` only for a deliberate legacy/migration setup. |
| `ADMIN_EMAILS` | Admin features | Comma-separated admin email allowlist. Normalize case/whitespace and match the Cloudflare Access policy. |
| `ACCESS_ISSUER` | Admin features | The team's Cloudflare Access team issuer URL. |
| `ACCESS_AUD` | Admin features | Audience tag for this Access application. |
| `TURNSTILE_SITE_KEY` | Public submissions | Public site key for the project's allowed hostname and `payment` action. |
| `SHEETS_ENABLED` | Optional; default `false` | Enable only after the Apps Script receiver and matching secret are configured and tested. |
| `SHEETS_WEBHOOK_URL` | Sheets enabled | Apps Script Web App `/exec` deployment URL. |
| `LINE_ENABLED` | Optional; default `false` | Enable only after a LINE bot, group, credentials, and webhook are configured. |
| `LINE_ADMIN_GROUP_ID` | LINE enabled | Destination group ID; treat as sensitive configuration. |
| `EMAIL_PROVIDER` | Always | `disabled` initially; choose `brevo` or `cloudflare` only after setup. |
| `EMAIL_FROM` | Email enabled | Verified sender address. |
| `ADMIN_EMAIL` | Email enabled | Destination address for operational notifications. |

Do not copy the current app's hostname, IDs, administrator addresses, or resource names. Replace them with values created for the new project.

### Cloudflare Worker secrets

| Name | Required when | Notes |
| --- | --- | --- |
| `TURNSTILE_SECRET_KEY` | Public submissions | Private secret paired with `TURNSTILE_SITE_KEY`; verify tokens server-side. |
| `SHEETS_WEBHOOK_SECRET` | Sheets enabled | Fresh random secret shared with Apps Script Script Properties; sign requests with HMAC. Never put it in a URL, Sheet cell, or browser code. |
| `LINE_CHANNEL_ACCESS_TOKEN` | LINE enabled | LINE Messaging API credential. |
| `LINE_CHANNEL_SECRET` | LINE webhook enabled | Verify incoming LINE webhook signatures. |
| `BREVO_API_KEY` | `EMAIL_PROVIDER=brevo` | Brevo API credential. |
| `GOOGLE_OAUTH_CLIENT_ID` | Legacy Drive adapter only | OAuth client used to access the owner's My Drive. Not required for private R2. |
| `GOOGLE_OAUTH_CLIENT_SECRET` | Legacy Drive adapter only | Keep with the legacy Drive OAuth credentials. |
| `GOOGLE_OAUTH_REFRESH_TOKEN` | Legacy Drive adapter only | Highly sensitive owner OAuth refresh token. Keep only while old Drive records need access. |

Never put secret values in Git, prompts, logs, issue descriptions, screenshots, or generated skill files. Generate unique secrets for each environment and rotate credentials if they are exposed.

### Local-only setting

| Name | Purpose |
| --- | --- |
| `LOCAL_ADMIN_TOKEN` | Development-only localhost fallback. Generate a random local value; do not configure it as a production Worker secret or use it as the real admin login. |

Keep `.dev.vars` ignored by Git. Commit only a sanitized `.dev.vars.example` with blank placeholders or harmless test keys.

### Bindings and Wrangler settings (not Worker secrets)

Create new resources and set project-specific names/IDs in `wrangler.jsonc` (or the selected Wrangler config):

- `DB`: D1 database binding, database name, and the new project's database ID. Apply all numbered SQL migrations before deploying code that depends on them.
- `SLIPS`: R2 bucket binding. Create a project-specific bucket and keep public access disabled; do not enable automatic deletion of payment evidence without a retention decision.
- `ASSETS`: static frontend directory binding if using Worker Static Assets.
- Cron trigger: enable the scheduled handler only if it is used for retry/reconciliation and monthly notifications. Confirm the time-zone logic in code rather than assuming the cron expression uses Bangkok time.
- `account_id`: deployment target identifier in Wrangler config or environment. It is not an application secret, but should be replaced with the new owner's account ID and must not be copied from this repo.
- Observability/logging settings: enable as appropriate, and ensure logs never contain tokens, uploaded file bytes, tenant contact data, or full slip URLs.
- Cloudflare deployment API token: configure only in the deployment environment/CI secret store, with the minimum required Worker, D1, and R2 permissions. It is not an application binding and must not be embedded in source files.

### Cloudflare Access and Turnstile dashboard

- Create an Access self-hosted application for the new production hostname and protect `/admin/*`, admin APIs, reports, vehicle-history endpoints, and private slip routes.
- Set the same Access issuer and application audience in Worker configuration. Worker-side JWT verification and `ADMIN_EMAILS` authorization are required even with Access enabled.
- Keep the Access allow policy and `ADMIN_EMAILS` in sync. A 403 after Access login means the authenticated email is not in the Worker allowlist; do not solve it by bypassing JWT validation.
- Create a Turnstile widget that allows the exact production hostname. Configure its public/secret keys and use the action `payment` consistently in the browser and Worker verification.

### Optional Google Sheets receiver

When enabling Sheets, create a **new private Sheet** and deploy the matching Apps Script receiver from `scripts/google-sheets-webhook.gs`:

- Script Properties: `SPREADSHEET_ID` and `SHEETS_WEBHOOK_SECRET`.
- Deploy as a Web App executing as the Sheet owner. If the endpoint must be reachable by the Worker without user OAuth, allow the Web App endpoint to receive requests and rely on the receiver's timestamped HMAC verification. The Sheet itself stays private.
- Set Worker `SHEETS_WEBHOOK_URL`, `SHEETS_WEBHOOK_SECRET`, then set `SHEETS_ENABLED=true` only after an authenticated test delivery succeeds.
- Keep D1 authoritative; Sheet edits do not flow back. Do not expose private slip images with public links or spreadsheet `IMAGE()` formulas.

### Optional LINE and email

- LINE: configure a bot, obtain a new access token and channel secret, add it to the intended group, record the group ID, set `LINE_ENABLED=true`, and register the webhook at `/api/webhooks/line`. Verify webhook signatures and test in a private group before production.
- Brevo: set `EMAIL_PROVIDER=brevo`, `BREVO_API_KEY`, a verified `EMAIL_FROM`, and `ADMIN_EMAIL`.
- Cloudflare Email Service: set up the sending domain/service and `send_email` binding named `EMAIL`; set `EMAIL_PROVIDER=cloudflare`, `EMAIL_FROM`, and `ADMIN_EMAIL`. Inbound Email Routing alone does not provide outbound sending.
- Keep both integrations disabled until every required field is configured and a test message is verified.

### Optional legacy Google Drive

New projects should use private R2. Only configure Drive if existing Drive-backed records must remain viewable or the owner explicitly selects Drive:

- Worker variable: `GOOGLE_DRIVE_ROOT_FOLDER_ID` (new project's root folder ID) and `GOOGLE_DRIVE_ADMIN_EMAILS` (emails allowed to access legacy files).
- Worker secrets: `GOOGLE_OAUTH_CLIENT_ID`, `GOOGLE_OAUTH_CLIENT_SECRET`, and `GOOGLE_OAUTH_REFRESH_TOKEN` from the Drive owner's OAuth flow.
- Verify the OAuth owner can access the root folder and restrict file sharing to admin accounts. Do not make files public. Do not remove these settings until all legacy records are migrated or no longer need retrieval.

## New project acceptance criteria

- No production-specific account IDs, project domains, bucket names, spreadsheet IDs, credentials, registry records, or receipt references remain from the source project.
- Clean migrations create the expected tables/indexes/triggers, and the database can be backed up and restored.
- Public vehicle lookup exposes only fields needed for payer selection; admin APIs, reports, history, and slip reads deny requests without a valid Access JWT and allowlisted email.
- A valid slip is stored in private R2 and linked to a D1 record; retries do not create duplicate records; review, rejection, and amount allocation preserve audit history.
- Receipt QR/status lookup is limited to the intended receipt data; the PDF renders and downloads without exposing admin-only information.
- Optional Sheets/LINE/email integrations remain off until their credentials and endpoints are configured; a failed integration does not corrupt the payment record.
- Local `.dev.vars` and CI/deployment credentials are not tracked by Git; setup docs list names and purposes, never values.
- Build, typecheck, and relevant automated tests pass before a requested production deployment.
