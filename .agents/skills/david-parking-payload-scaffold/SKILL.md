---
name: david-parking-payload-scaffold
description: Scaffold a parking-rental payment and slip-record system using Payload CMS on Cloudflare Workers, D1, and private R2. Use as the Payload alternative to the Worker-only scaffold; do not use for unrelated applications.
metadata:
  short-description: Scaffold parking records with Payload CMS
---

# David Parking scaffold with Payload CMS

Use this skill to create a new project with the same parking-payment scope as `david-parking-scaffold`, using Payload CMS for the admin panel, collections, access rules, and APIs. Treat the source project as an architectural reference only. Never copy its production identifiers, secrets, administrator/customer records, or deployment state.

## Start from the Cloudflare Payload template

Use Payload's official `with-cloudflare-d1` template as the runtime base, then adapt it to the parking requirements below. Do not start from a generic Node host configuration and assume it will run unchanged in Workers.

The current official template uses Payload v3 with Next.js App Router/React, TypeScript, OpenNext for Cloudflare Workers, Payload's D1 SQLite adapter, the R2 storage adapter, and Wrangler. It binds D1 as `D1` and R2 as `R2`; the generated Worker and assets are built under `.open-next/`. Follow the current template for exact versions, compatibility flags, runtime entrypoint, and build scripts instead of pinning old versions from this skill.

Cloudflare's current Payload template notes a Paid Workers requirement due to bundle-size limits. Check the current template and plan limits before choosing the deployment plan; keep an eye on Worker bundle size as features are added. Do not target Cloudflare Pages unless the owner explicitly chooses a separately verified Pages architecture.

## Preserve the product and security rules

- Keep the plate-centric model: license plate identifies a vehicle, while slot, tenant, rent, and status may change over time. Preserve effective-dated history and payment-time snapshots.
- D1 is the system of record. Store money as integer satang, preserve audit events, and enforce critical invariants in database constraints/migrations as well as application logic.
- Store new slip images in a private R2 bucket. Disable public bucket access and `r2.dev`; only serve files through an authenticated admin route. Do not copy the template Media collection's public-read access setting for payment evidence.
- Keep Cloudflare Access as the admin perimeter and retain the second server-side administrator allowlist (`ADMIN_EMAILS`). Validate the signed Access JWT at protected routes: signature, issuer, audience, expiry, then normalized email allowlist. Payload collection access controls must also deny unauthorized operations. Never trust a client-supplied email or assume Access automatically signs the user into Payload.
- Decide and document the admin sign-in experience. The Cloudflare Access gate and Payload's auth-enabled Users collection are separate by default. If both are required, implement an explicit identity mapping or accept and document the second Payload login; do not claim automatic SSO without implementing it.
- Keep payer APIs public only where required. Put payer endpoints on a route namespace explicitly handled separately from Payload's REST catch-all (for example, custom Payload endpoints under `/api/public/*`); guard them with server-verified Turnstile, validation, rate limits, file checks, and idempotency. Protect Payload admin and private data APIs with Access plus Payload authorization without accidentally blocking payer routes.
- A submitted slip is provisional evidence, not confirmed bank settlement. Retain duplicates for review; support approve/reject and manual allocation for partial, extra, penalty, or other amounts.
- Provide a receipt/reference with a limited QR status page and downloadable/printable PDF. Never include private slip URLs, admin information, or tenant personal data in the QR response.
- Google Sheets, LINE, and email are optional. Keep them disabled initially; D1 remains authoritative and Sheets is a one-way mirror. The default D1/R2 architecture needs no Google account, OAuth, Service Account, or Google API credentials. If Sheets is enabled, its generated shared HMAC secret is an application signing secret, not a Google credential.
- Use Bangkok time in the UI and `YYYY-MM` rental months. Store timestamps in UTC.

## Payload domain model and implementation guidance

Create Payload collections and custom endpoints/hooks around the same domain entities. Adapt field names to the Payload version, but preserve these responsibilities:

- `users`: auth-enabled administrators with explicit active status and role/access rules.
- `tenants`: name, phone, email, LINE ID, note, status; all private and admin-only.
- `vehicles`: unique license plate, province, tenant relation, current slot, status, rental start/end, current fee, revision.
- `vehicle-terms`: effective month, slot, tenant, and fee history. Do not overwrite old terms when a car moves or its rent changes.
- `payment-submissions`: idempotency/fingerprint, plate and slot/fee snapshots, month, total/unallocated satang, review/allocation state, slip reference, and lifecycle state.
- `payment-logs` / allocations: per-month allocated satang, review status, reviewer, and immutable payment-time values.
- `audit-events`, `notification-deliveries`, and `sheet-sync`: appendable history and retryable integration state with idempotent event keys.
- A dedicated `payment-slips` upload collection or private storage adapter for evidence. Keep its read access private and route file reads through authorization checks.

Payload's collection access functions are the default authorization layer for admin CRUD, but they are not a replacement for row-level financial invariants. Use Payload migrations and SQLite/D1-supported constraints for uniqueness and critical validation. Payload documents SQLite transaction limitations; explicitly verify atomic behavior for payment commit, allocation, audit, and notification scheduling. Use a supported D1 batch/transaction boundary or redesign the operation if the selected adapter does not guarantee atomicity. Do not assume a series of Local API calls is one transaction.

Use Payload Local API for ordinary server-side CRUD and access-controlled collections. Use custom Payload endpoints or Next.js route handlers for payer workflows, receipts, reports, and provider webhooks where a workflow spans multiple collections or needs a strict response shape. Validate and authorize at the server boundary; never pass privileged Local API context to untrusted callers.

Use Payload's migration commands to generate and review schema migrations. Apply migrations to local/test D1 first and then production before the corresponding code deploy. Never use development schema-push behavior as a production migration plan. Keep migrations additive and preserve legacy data until a verified migration plan says otherwise.

For R2 uploads, use the official `@payloadcms/storage-r2` adapter when it fits the selected Payload release and Worker bindings. Verify the actual generated file URLs and access behavior: the bucket stays private, and slip retrieval must use an authorized route. If the adapter's public media behavior does not meet this rule, use a custom storage/download path rather than making the bucket public.

The official template currently disables image crop/focal-point features on Workers because of runtime limitations. Check the latest template before enabling image transforms or adding native modules such as `sharp`.

## Scaffold workflow

1. **Inspect and isolate.** Review repository instructions, current Payload Cloudflare template, requirements, local migrations, and deployment config. Never read/copy a live `.env`, `.dev.vars`, secret store, production D1 records, customer registry, or live resource IDs. Work only in the explicitly requested new project directory.
2. **Confirm decisions.** Record project name/domain, initial registry input, rent and slot rules, administrators, receipt retention, expected load, and optional integrations. Ask about choices that materially affect data ownership, admin sign-in, storage privacy, or payments.
3. **Create project identity.** Replace template Worker/database/bucket names and IDs, hostnames, branding, email labels, and deployment settings with new-project values. Use separate D1 and R2 resources for development/staging/production; never point local development or preview builds at production data.
4. **Model the domain.** Define Payload collections, access controls, generated types, D1 migrations, unique keys, audit records, and payment invariants. Keep one unique vehicle identity per plate and history for time-varying slot/fee terms.
5. **Build workflows.** Implement the public three-step payer form, private slip upload/read, receipt status/QR/PDF, admin dashboard, tenant contact/note management, CSV import/export with preview, payment review/allocation, reports with month/year filters, and retryable optional integrations.
6. **Configure Cloudflare.** Set D1/R2 bindings and Worker variables/secrets below; configure Cloudflare Access policies, Turnstile hostname/action, production domain, cron/scheduled handler, logs, and backups. Make sure the Cloudflare Pages/Workers choice and plan meet Payload's current template and bundle requirements.
7. **Verify before go-live.** Run typecheck, build, and project tests. Verify unauthorized admin/API/file access is denied, public payer flows work with valid Turnstile, private R2 remains private, migrations are applied, duplicate/idempotent payment behavior is correct, allocation totals are enforced, receipts/QR do not leak data, and disabled integrations do not block payment intake. Deploy only when deployment is explicitly requested; do not import production records or mutate live resources without authorization.

## Configuration checklist

Use the current official template's names for Payload/Cloudflare-specific settings. The names below match the current template where stated; keep custom feature names consistent across Worker bindings, `payload.config.ts`, and Next route handlers. Put local values in ignored `.env`/Wrangler dev vars and production values in Cloudflare secrets/vars. Commit only sanitized examples.

### Required secret

| Name | Purpose |
| --- | --- |
| `PAYLOAD_SECRET` | Long random secret required by Payload for auth/session signing and related secret operations. Generate a fresh value per environment; keep it out of Git and build output. |

### Admin and public form settings

These are custom settings to implement consistently; only create those used by the project:

| Name | Type | Purpose |
| --- | --- | --- |
| `ACCESS_ISSUER` | Variable | Cloudflare Access team issuer used for server-side JWT verification. |
| `ACCESS_AUD` | Variable | Audience for the Access application protecting this project. |
| `ADMIN_EMAILS` | Variable | Comma-separated second admin allowlist; must match the Access policy after normalization. |
| `APP_ORIGIN` | Variable | Exact public origin for origin checks, receipts, and callbacks. |
| `TURNSTILE_SITE_KEY` | Variable | Public widget key for the production hostname and `payment` action. |
| `TURNSTILE_SECRET_KEY` | Secret | Private server-side key paired with the site key. |

### Optional integration settings

| Name | Type | Enable only when |
| --- | --- | --- |
| `SHEETS_ENABLED` | Variable | `true` only after testing the Apps Script receiver. Keep `false` by default. |
| `SHEETS_WEBHOOK_URL` | Variable | Apps Script `/exec` endpoint for the one-way Sheet mirror. |
| `SHEETS_WEBHOOK_SECRET` | Secret | Fresh random HMAC key shared with Apps Script Script Properties; not a Google login/API credential. |
| `LINE_ENABLED` | Variable | `true` after LINE setup. Keep `false` by default. |
| `LINE_CHANNEL_ACCESS_TOKEN` | Secret | LINE Messaging API token. |
| `LINE_CHANNEL_SECRET` | Secret | LINE webhook signature validation secret. |
| `LINE_ADMIN_GROUP_ID` | Variable | Target admin group. Treat as sensitive configuration. |
| `EMAIL_PROVIDER` | Variable | `disabled` initially; select supported provider only after setup. |
| `BREVO_API_KEY` | Secret | Required only when using Brevo. |
| `EMAIL_FROM` | Variable | Verified sender when email is enabled. |
| `ADMIN_EMAIL` | Variable | Notification recipient when email is enabled. |

### Non-secret bindings and deployment settings

- Cloudflare D1 binding named `D1`, connected to a new per-environment database; Payload's D1 adapter uses this binding, not a database connection string.
- Cloudflare R2 binding named `R2`, connected to a private bucket; disable public access and `r2.dev`.
- Static asset binding/config and Worker entrypoint generated by the current official OpenNext Cloudflare template; do not replace `.open-next/` settings with the Worker-only Vite layout.
- `nodejs_compat` and other compatibility flags from the template. Refresh the compatibility date/flags from current documentation during setup.
- Optional `NEXT_INC_CACHE_R2_BUCKET` only if the selected OpenNext template/version requires persistent Next cache in R2.
- Cloudflare account ID and D1/R2 resource IDs in Wrangler config are project-specific deployment identifiers, not secrets; never copy them from the reference repository.
- Cloudflare API token for CI/deploy is stored in CI's secret manager with minimum required permissions, not in Payload/Worker source or runtime app vars.
- Local-only `LOCAL_ADMIN_TOKEN` is not required by Payload and must not become a production login mechanism.

### Cloudflare Access, R2, and optional Sheets

- Protect the Payload Admin UI and sensitive Payload/custom APIs with Cloudflare Access. Preserve public payer endpoints explicitly; do not accidentally gate or expose them through broad path rules.
- Validate the signed Access JWT on the server/Worker boundary for protected operations. Keep Access policy and `ADMIN_EMAILS` in sync, and also enforce Payload Users roles/access controls. Do not rely only on hiding the Admin UI.
- Keep R2 private. Verify unauthorized users cannot fetch slip objects by guessed keys or generated URLs. Public marketing/media images, if any, must use a distinct collection/bucket policy from payment evidence.
- If Sheets is enabled, Apps Script Script Properties contain `SPREADSHEET_ID` and the same generated `SHEETS_WEBHOOK_SECRET`; the Sheet itself remains private. The Worker sends signed, minimal snapshots, not slip bytes or tenant contact details.
- Google Sheets does not require Google OAuth or Service Account credentials in the Worker. Do not add Google account credentials to the baseline setup.

## Acceptance criteria

- The new project is based on the maintained Payload Cloudflare D1 template and has no copied account IDs, production hostnames, bucket/database IDs, credentials, customer records, or receipts.
- Payload Admin and collections provide the admin workflows; public payer endpoints remain distinct and do not expose admin or tenant data.
- Access JWT verification, `ADMIN_EMAILS`, Payload access controls, and private R2 file reads all deny unauthorized requests when tested directly.
- D1 collections/migrations preserve plate identity, effective-dated terms, immutable payment snapshots, integer-satang totals, audit history, and safe duplicate/idempotent retries.
- Critical payment and allocation writes have a verified atomic boundary under the actual D1 adapter; a partial failure cannot create a false paid status or lose the uploaded slip.
- Receipt QR/PDF works and exposes only limited receipt status. CSV workflows and reports support the required filters.
- Sheets/LINE/email are disabled by default, and failed integration delivery does not corrupt or reject a committed payment.
- Local/CI secrets are not tracked, backups and environment isolation are documented, and typecheck/build/tests pass before a requested production deployment.

## Current upstream references

Recheck these before using the skill because versions, Workers compatibility, limits, and template settings change:

- [Payload Cloudflare D1 template](https://github.com/payloadcms/payload/tree/main/templates/with-cloudflare-d1)
- [Payload SQLite/D1 adapter](https://payloadcms.com/docs/database/sqlite)
- [Payload storage adapters and R2](https://payloadcms.com/docs/upload/storage-adapters)
- [Payload production deployment](https://payloadcms.com/docs/production/deployment)
