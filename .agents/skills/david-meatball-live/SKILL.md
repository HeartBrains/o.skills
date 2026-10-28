---
name: david-meatball-live
description: Maintain, deploy, or extend David Meatball Live, a Thai daily stock and ordering system on Cloudflare Workers and D1 with an admin dashboard, public write-only form, reports, and protected MCP. Use together with serverless-nocms for full domain rules.
metadata:
  short-description: David Meatball inventory system
---

# David Meatball Live profile

Read [serverless-nocms](../serverless-nocms/SKILL.md) and its system blueprint before implementing domain changes. This profile captures project-specific choices while keeping deploy IDs, domains, data, and credentials out of O.skills.

## Project architecture

- Cloudflare Worker with Hono handles static assets, REST API, Cloudflare Access verification, D1 service operations, optional notifications, and an optional MCP Streamable HTTP endpoint.
- Cloudflare D1/SQLite is the system of record. Migrations are additive and report lines keep snapshots so later catalog edits do not rewrite history.
- Frontend is React + TypeScript + Vite. The Worker serves built static assets; the public form and admin app share validated API/service logic where appropriate.
- The public `/submit` form is intentionally write-only. Admin dashboard, reports, product catalog, historical records, and MCP require both Cloudflare Access and Worker-side JWT/issuer/audience/email allowlist validation.
- MCP is disabled by default until its client identity flow is proven. MCP tools call the same authorization, validation, audit, and service layer as the web app; never expose SQL or raw D1.

## Business invariants

- Record remaining stock and manually entered order bags. Do not auto-calculate order quantities.
- Preserve Thai labels and any user-provided Myanmar labels exactly; do not translate or normalize those values without instruction.
- Keep product recommended bags and sticks/items per bag editable as catalog metadata, with per-report snapshots.
- Treat actual daily sales as an optional override of estimated sales revenue. Clearly label calculations as estimates and keep stock movement variance distinct from sales.
- Capture sale price and purchase price snapshots for reports; changing today's catalog prices must not alter earlier reports. Cost-only complimentary products have no standalone sale price.
- Bangkok dates are used for user-facing report dates. The default next date skips Sunday but remains editable.
- Public form validation, duplicate guard, rate limiting, and write-only access are enforced at the Worker, not only in the UI. Notification failures do not undo saved reports.

## Project operations

- Before changes, inspect current `spec.md`, README, migrations, API/services, Wrangler config, and CI workflow. Current code/config can evolve; treat project files as authoritative over this profile.
- Run the repo's build, typecheck, local D1 migrations, and relevant auth/API checks before a release. Use local/test data for verification.
- Never run destructive SQL, replace production history, import unreviewed data, change Access policy, or deploy unless the user asks for that action.
- Create fresh Cloudflare resources for a new deployment; never copy a live account ID, database ID, Access audience, hostname, allowlist, or token from this profile or another environment.
- Keep notification integrations optional. Store Resend/LINE and Cloudflare deployment credentials in the appropriate secret manager. This skill intentionally records names only in its referenced setup guide, not values.
- For deployment, verify the intended Worker, D1 binding, migration state, Access app, public form exception, and target hostname before applying changes.

## Related skills

- [serverless-nocms](../serverless-nocms/SKILL.md) — complete reusable application specification, data model, API, MCP design, and Cloudflare setup.
- [llm-auto-cms-google-docs-serverless](../llm-auto-cms-google-docs-serverless/SKILL.md) — only for a future Google Sheets/Docs content integration; Google is not the system of record in this project.
