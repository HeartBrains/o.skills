---
name: serverless-nocms
description: Plan, scaffold, or rebuild a production-ready serverless NoCMS inventory and daily ordering system on Cloudflare Workers and D1, with a Thai admin UI, public stock form, reporting, optional MCP, and operational setup. Use for this David Meatball-style system, not for unrelated serverless applications.
metadata:
  short-description: Build the Serverless NoCMS stock system
---

# Serverless NoCMS project skill

Use this skill to create a new instance of the daily stock and order-record system described in the references. “NoCMS” means the product catalog and workflows are implemented in the application itself; do not add a third-party CMS. D1 is the system of record.

## Read the relevant references

- Read [system-blueprint.md](references/system-blueprint.md) for product behavior, data model, APIs, MCP tools, calculations, and acceptance criteria.
- Read [cloudflare-setup.md](references/cloudflare-setup.md) when preparing local, Cloudflare, Access, MCP, email, LINE, or deployment configuration.
- Read [catalog-reference.md](references/catalog-reference.md) only when the user wants to reuse or adapt the David Meatball product labels and packaging baseline.

## Preserve the domain rules

- Use Cloudflare D1 as the authoritative store; keep reports, product metadata, audit events, and historical snapshots in the database.
- The main workflow records daily remaining stock and manually entered quantities to order. Never calculate or auto-submit an order quantity.
- Support Thai as the primary UI language and preserve user-provided Myanmar labels exactly. Do not invent translations.
- Preserve old report snapshots when product names, packaging, units, or prices change.
- Use `Asia/Bangkok` for dates shown to users and UTC timestamps for stored events. Default the next date to the next business day by skipping Sunday, while allowing an authorized editor to change it.
- Treat actual sales as an optional entered value that overrides the estimated sales amount for that day. Mark calculated sales, revenue, cost, and gross profit as estimates.
- The complimentary sauce is cost-only: it has no separate sale price, and its cost is included when ordered.
- Make the public form write-only. Protect dashboards, admin APIs, report history, product management, and MCP at the Worker boundary.
- MCP must call the same validated application services as the web UI, log tool actions, and never expose arbitrary SQL or direct D1 access.
- Optional email/LINE notification failure must not undo a successfully saved public report.

## Scaffold workflow

1. Inspect the requested destination repository, existing instructions, and current files. Keep work inside the target project and preserve unrelated changes.
2. Establish whether this is the same shop or a different business. Confirm the new project name/domain, allowed admin identities, catalog seed, current sale/purchase prices, MCP client/auth approach, and which optional notifications to enable. Do not block on choices that can safely remain disabled.
3. Write or update a project specification before implementation when the user asks for planning or when the scope is not yet agreed. For a clear implementation request, proceed with the agreed scope.
4. Build the Worker API, D1 migrations, React/Vite admin app, static public form, reports, and MCP in coherent vertical slices. Keep SQL parameterized and migrations additive.
5. Add project-specific names and fresh Cloudflare resource IDs. Use the setup reference; never copy production credentials, account IDs, D1 IDs, Access AUD values, domains, report records, or `.dev.vars` from the reference instance.
6. Run typecheck/build and the meaningful migration/API/auth checks available in the target repo. Verify public and admin boundaries, mobile layouts, date/duplicate behavior, snapshots, report calculations, PDF generation, and MCP authorization.
7. Deploy only when the user requests deployment. Before a live deploy, confirm that the target account, domain, D1 binding, and migration state belong to the new project; do not run destructive SQL or import production history without explicit scope.

## Security and configuration rules

- Never place credential values in the skill, source, committed examples, prompts, logs, screenshots, or generated documentation.
- Commit only sanitized examples with blank values. Store production values as Cloudflare Worker secrets/variables and CI deployment credentials in the CI secret manager.
- Use Cloudflare Access plus Worker-side signature, issuer, audience, expiry, and admin allowlist verification. Match Access policy emails with the Worker `ADMIN_EMAILS` list.
- Do not assume a Cloudflare Access Service Token works as a user JWT. If an MCP client cannot obtain/refresh a supported identity token, define a separate authenticated machine-client design before enabling MCP; do not bypass the Worker allowlist.
- Keep optional integrations disabled until their credentials and recipients are configured and tested. See [cloudflare-setup.md](references/cloudflare-setup.md).

## Sample invocation

> Use `$serverless-nocms` to build this daily stock and ordering system in `<target-repo>`. Start with a specification, use the Cloudflare Workers + Hono + D1 + React/Vite architecture, include the public write-only form, admin workflows, reporting, and protected MCP described in the skill references. Use fresh project-specific Cloudflare resources, leave notifications disabled until configured, and do not deploy until I ask.
