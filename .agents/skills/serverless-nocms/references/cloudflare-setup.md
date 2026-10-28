# Cloudflare and integration setup

This checklist is for a new instance. Resource names and IDs must be created for that instance; never copy the live David Meatball identifiers.

## Cloudflare resources

Create a project-specific:

- Worker with a unique name and a custom domain when ready.
- D1 SQLite database bound as `DB`.
- Static assets binding named `ASSETS`, pointing to the Vite build output.
- Rate Limit binding named `PUBLIC_FORM_LIMITER` (reference behavior: five public submissions per minute per client IP).
- Cloudflare Access self-hosted application for the admin hostname and admin/MCP paths.
- Access policy allowing only the intended administrator emails. Add a narrowly scoped Bypass policy for `/submit` only if the public form must bypass Access.

Run additive D1 migrations against local D1 first. Confirm the remote database name/ID and migration history before applying production migrations. Set a compatibility date suitable for the new Worker and keep `nodejs_compat` only if dependencies require it.

## Worker variables

Set only values needed by implemented features. These are names/placeholders, not credentials:

| Name | Type | Purpose |
| --- | --- | --- |
| `ENVIRONMENT` | Variable | `development` locally; `production` in Cloudflare. |
| `ADMIN_EMAILS` | Variable or Secret | Comma-separated allowed admin email addresses; match the Access allow policy. Treat the list as private configuration. |
| `CF_ACCESS_ISSUER` | Variable | Cloudflare Access team issuer URL, e.g. `https://<team-name>.cloudflareaccess.com`. |
| `CF_ACCESS_AUD` | Variable | Application Audience tag for this project's Access app. |
| `MCP_ENABLED` | Variable | Keep `false` until MCP authentication is confirmed; then set `true`. |
| `NOTIFICATION_FROM_EMAIL` | Variable | Verified sender address for Resend email notices. |

`DEV_ADMIN_EMAIL` is local-only and must not substitute for production authentication. Do not configure `ADMIN_EMAILS` or Access policy from this reference project's live values.

## Worker secrets

Create fresh credentials in the appropriate provider; do not paste their values into Git or this skill.

| Name | Required when | Provider/setup |
| --- | --- | --- |
| `RESEND_API_KEY` | Email notices enabled | Resend API key with send permission. Verify the sending domain/address and set `NOTIFICATION_FROM_EMAIL`. |
| `LINE_CHANNEL_ACCESS_TOKEN` | LINE notices enabled | LINE Developers > Messaging API channel > issue a channel access token. |
| `LINE_TARGET_ID` | LINE notices enabled | Destination user/group/room ID; ensure the bot can message that destination. Treat as configuration secret. |
| `CLOUDFLARE_API_TOKEN` | CI or remote deployment | Store only in CI's secret manager or approved local secret store, with least-privilege Worker/D1 permissions. Never expose it to frontend/runtime code. |

The app sends email through Resend to `ADMIN_EMAILS` and LINE push messages to `LINE_TARGET_ID`. Each channel is skipped when its required configuration is absent. Failed delivery is logged without exposing tokens and does not roll back D1 writes.

## Bindings and non-secret deployment identifiers

- `DB`: D1 database binding and the newly created database ID in Wrangler config.
- `ASSETS`: built static assets directory.
- `PUBLIC_FORM_LIMITER`: rate-limit binding with a project-unique namespace ID.
- Worker name, account ID, D1 ID, Access AUD, custom domain and rate-limit namespace are project-specific. They are not secret values, but they must not be copied from the reference deployment.

## Access and Worker authentication

The Worker validates the signed Cloudflare Access JWT: signature, issuer, audience, expiry and normalized email allowlist. The request must carry the app token in `Cf-Access-Jwt-Assertion` or `Authorization: Bearer <JWT>`. Cloudflare Access policy and `ADMIN_EMAILS` must authorize the same people.

Do not confuse these mechanisms:

- Browser users authenticate interactively through Cloudflare Access.
- A static Cloudflare Access Service Token does not include the user email expected by the reference Worker allowlist and therefore is not a drop-in MCP credential.
- Gemini/API MCP clients that support custom request headers can send an approved Access JWT, but access JWTs expire. Do not paste one into Git, documentation, or chat. If a client needs durable unattended access, implement a dedicated OAuth or machine-identity flow with explicit allowlisting, rotation, and audit before enabling MCP.

## Local setup

1. Install the repository's pinned dependencies and copy `.dev.vars.example` to ignored `.dev.vars`.
2. Fill only local test values. Keep all blanks in the committed example; do not store production tokens locally.
3. Apply local migrations and run Worker/UI separately as documented in the new repo's README.
4. For local admin flow, set `DEV_ADMIN_EMAIL` to a development address present in local `ADMIN_EMAILS`; never enable this fallback in production.

## Production rollout checklist

1. Build and typecheck. Apply reviewed D1 migrations; preserve a backup/restore point before any schema/data transformation.
2. Set Access issuer/AUD and the admin allowlist. Confirm an authorized browser user can pass both Access and Worker validation and a non-admin is denied.
3. Verify `/submit` is intentionally public and write-only while `/api/*` and `/mcp` remain protected. Confirm Access path ordering does not bypass any admin route.
4. Configure optional notifications only after sender/recipient setup. Submit one controlled test record or use a safe test environment; verify email and LINE independently.
5. Deploy only after explicit user instruction. Smoke-check the intended host, public form, protected API, migration state, and logs; do not print tokens or full authorization headers.

## Required repository files

- Sanitized `.dev.vars.example` with empty values/comments only.
- `.gitignore` entry for `.dev.vars`, local databases, build artifacts as appropriate, and credential files.
- Wrangler config with project-specific placeholders populated only after resource creation.
- README setup guide listing names and dashboard paths, not secret values.
- CI workflow that reads deployment tokens from repository/org secrets and runs build/typecheck before deploy.
