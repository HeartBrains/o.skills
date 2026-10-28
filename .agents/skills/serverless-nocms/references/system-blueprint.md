# System blueprint

This document describes the application to recreate. It is a business-system specification, not a copy of the live deployment or its credentials.

## Purpose and scope

Build a Thai-language daily inventory and ordering application for a small food shop selling skewered meatballs, sausages, processed foods, and supporting supplies. Product names may also carry Myanmar labels for staff. The app replaces the data-entry and reporting workflow with a custom NoCMS application.

### In scope

- Public daily stock/order form, limited to creating a new report.
- Admin dashboard, report list/detail/edit, product catalog, sales/cost summaries, daily stock detail, CSV export, printable/shareable order text, and PDF preview/download.
- Optional Resend email and LINE Messaging API notifications after a public submission.
- Optional protected MCP endpoint for the same read/create/update operations available to the admin.
- Optional legacy import module with preview, row-level outcomes, and duplicate handling when the user requests migration.

### Out of scope unless separately requested

- Customer-facing product sales/orders, POS transactions, automatic stock deduction, vendor purchase/receiving workflows, automatic reorder calculations, bookkeeping/tax filing, and bidirectional Google Sheets synchronization.
- Do not invent approval stages. The user enters order quantities directly.

## Technology architecture

```text
React + TypeScript + Vite (responsive Thai admin UI)
                 │ HTTPS JSON / MCP Streamable HTTP
                 ▼
Cloudflare Worker + Hono
  ├─ Cloudflare Access JWT verification and admin allowlist
  ├─ REST API, validation, reporting and audit services
  ├─ Optional protected MCP tools
  ├─ Public write-only form and optional notifications
  └─ Vite static assets
                 │ D1 binding
                 ▼
Cloudflare D1 (SQLite; authoritative application data)
```

Use TypeScript, Hono, D1 prepared statements, Wrangler, React, Vite, and Lucide-style icons. Keep the frontend static and serve it through the Worker assets binding. Use a mobile-first responsive UI. Do not introduce a CMS or ORM without a project-specific reason.

## Roles and authorization

- **Public submitter:** may load the public form and create one daily report. Cannot read reports, list the catalog through a separate API, edit records, or access admin routes.
- **Admin:** authenticates through Cloudflare Access and must also pass Worker-side JWT verification and `ADMIN_EMAILS` allowlisting. May manage products, reports, summaries, exports, and MCP operations.
- A system may start with one admin, but configuration should permit a comma-separated allowlist where needed.
- Public form access is a deliberate exception in Cloudflare Access for `/submit`; every admin/API/MCP path remains protected.

## Business rules

### Products and units

- Product records contain a stable ID, source/order number, display order, Thai name, Myanmar label, stock unit, purchase unit, recommended bags, items per bag, sale price, purchase price, effective/as-of date for purchase price, cost mode, stock-field flag, active flag, and optimistic version.
- Admins can add/edit/deactivate products, change units/labels/packaging recommendations and update sale/purchase prices. Inactive items are visually distinct and excluded from the new public form.
- The public/admin report captures stock remaining in the product's stock unit and order quantity in its purchase unit (usually bags). Show recommended bag count and sticks-per-bag as a compact hint; never calculate the amount to order.
- Changes to catalog metadata do not alter historical report-line snapshots.
- The complimentary sauce is `ORDER_COST_ONLY`: no separate sales price; recognize its purchase cost on the report where it is ordered.

### Daily reports

- A report contains `reportDate`, editable `nextDate`, `orderStatus` (`ORDER` or `NO_ORDER`), `recorderName`, optional `actualSalesAmount`, and inventory lines (`productId`, `remainingQuantity`, `orderQuantity`).
- Use Bangkok calendar dates. Default `nextDate` to the next day excluding Sundays; validate that it is after `reportDate`. The public form may submit today or an earlier date, never a future date.
- There is one report per `reportDate`. Use a unique database constraint and idempotency key. Duplicate public submissions receive a clear conflict response; never silently overwrite an existing report.
- Require at least one quantity. Validate IDs against the current active catalog, reject negative values and malformed/oversized payloads, and retain the report plus audit record atomically.
- Admin editing uses optimistic `version` checks, writes an audit event, and preserves snapshots for each inventory line.

### Sales, cost, and stock reports

- Estimate sold quantity per product/day as prior remaining quantity plus incoming quantity from the prior report's order (only when its `nextDate` has arrived) minus current remaining quantity.
- Convert bags to stock units only when a valid `itemsPerBag` snapshot exists. Do not sum quantities across unlike units. If a prior snapshot, current stock, or conversion is missing, mark the line uncalculated instead of guessing.
- Display negative movement separately as inventory increase/variance; never show it as negative sales.
- If a daily `actualSalesAmount` exists, use it instead of estimated sales revenue for that day's total. Keep the underlying item-level quantity estimate distinct and label it clearly.
- Estimate revenue using the sale-price snapshot per stock unit. Estimate purchase cost using purchase-price snapshots and order quantity per purchase unit. Gross profit is estimated revenue less estimated costs; omit unpriced/incomparable lines from currency totals and show a completeness indicator.
- Group sauce/cost-only items into order costs without a standalone sale amount.
- Filters support day, month, year, and custom date range. Default daily view to today, current range controls to the current month, and stock detail to the current month. Daily summary should link directly to editing that report and allow previous/next-day navigation.
- Show remaining stock and ordered quantity as separate columns/values for each date.

### Public form and notifications

- `GET /submit` renders current active product labels/units and recommended packaging hints; `POST /submit` is write-only.
- Default report date to today and next date to the next non-Sunday business day; allow explicit edits. Include submitter name, order status, remaining quantity, and manually selected order bags (quick choices 1–5 plus editable numeric input).
- Apply honeypot validation, request JSON/content-type validation, a hard body-size limit around 128 KB, and a rate limit around five requests per minute per IP. Reject empty, stale-catalog, future-date, invalid-date, or duplicate submissions.
- After a successful commit, send optional email via Resend to `ADMIN_EMAILS` and optional LINE push to the configured user/group/room. Run notifications in the background and isolate failures from the committed report. Do not send secrets or unnecessary personal data.

### Admin experience and exports

- Dashboard shows latest reports and operational totals; cards/summary metrics should filter or navigate to the corresponding list when clicked.
- Report list supports date/status filters, search as appropriate, sorting, and a practical page size. Detail pages show stock/order lines and edits.
- Product list supports active/inactive filters, search, direct status toggles, compact mobile cards, and inline/edit views that avoid horizontal clipping.
- Provide Thai UTF-8 CSV export for reports.
- For a selected day's order, provide a clean text export with date, product, quantity, and unit (no receiving-date field), plus Web Share API sharing where available and copy/download fallback.
- Sales summary PDF opens a preview, is A4 portrait, supports multiple pages, and provides a separate save/download control. Design print styles for portrait page width and legible scaling.

## D1 logical model

Use additive numbered migrations, foreign keys, indexes, bound parameters, and a local/remote migration workflow.

### `products`

Core fields: `id`, unique `source_number`, unique `sort_order`, `name_th`, `label_my`, `stock_unit`, `purchase_unit`, `recommended_bags`, `items_per_bag`, `sale_price`, `purchase_price`, `purchase_price_as_of`, `cost_mode` (`SALES` / `ORDER_COST_ONLY`), `has_stock_field`, `is_active`, `version`, `created_at`, `updated_at`.

### `daily_reports`

Core fields: `id`, unique `idempotency_key`, unique `report_date`, `next_date`, checked `order_status`, `recorder_name`, nullable `actual_sales_amount`, `created_by`, `version`, `created_at`, `updated_at`.

### `inventory_lines`

One row per product/report; store nullable `remaining_quantity` and `order_quantity`, product ID, and snapshots for Thai/Myanmar name, stock/purchase units, recommended bags, items per bag, sale/purchase prices, purchase-price date, and cost mode. Unique `(report_id, product_id)`.

### `audit_events` and `mcp_events`

Record actor, action/tool, entity, result, timestamp, and limited change metadata. Do not log credentials, full prompts, or unnecessary submitted data.

### `legacy_import_rows` (optional module)

If legacy import is requested, record unique source row identity, source row number/date, import status (`IMPORTED`, `DUPLICATE_IGNORED`, `EXISTING_REPORT_KEPT`), destination report, duplicate reference, and raw source JSON for traceability. Build preview/validation before writes and preserve original reports.

## API surface

Implement equivalent behavior with documented validation and response shapes. Admin routes require the Worker auth middleware; public exceptions are only the form and explicitly approved health route.

- `GET /health` — service health response.
- `GET /submit`, `POST /submit` — public form and create-only submission.
- `GET /api/session`, `/api/dashboard`, `/api/sales-summary`, `/api/stock-summary`, `/api/inventory/current`.
- `GET/POST /api/products`, `PATCH /api/products/:id`, `GET /api/products/:id/history`.
- `GET/POST /api/reports`, `GET /api/reports/:id`, `PATCH /api/reports/:id`, `POST /api/reports/:id/submit`.
- `GET /api/orders`, `GET /api/orders/:id`, `GET /api/export.csv`.
- `POST/PATCH` writes must use shared domain service functions, schema validation, audit events, and revision checks.
- Return no-store/private headers for protected API responses. Use explicit 400/401/403/404/409/413/415/429/500 responses with user-safe error messages.

## MCP surface

Use the official MCP TypeScript server SDK in the Worker with Streamable HTTP transport. Protect `POST/GET/DELETE` MCP traffic using the same Worker admin middleware. Keep `MCP_ENABLED=false` until a compatible client auth design is verified.

Reference tools:

- `list_products` (optionally include inactive items)
- `create_product`, `update_product`
- `list_daily_reports`, `get_daily_report`
- `create_daily_report`, `update_daily_report`

Every tool validates with the same Zod/service rules as REST, records an MCP audit event, and returns structured/text data. Do not offer SQL, arbitrary D1, hard-delete, or unaudited mutation tools. Identity must map to an allowlisted admin. If a client cannot obtain/refresh Cloudflare Access user JWTs, define and validate a dedicated machine-auth design before enabling MCP.

## Acceptance criteria

- New project has fresh names, hostname, D1 database, rate-limit namespace, Access app and credentials; no production identifiers or records copied.
- Local migrations apply cleanly and preserve uniqueness, FKs, snapshots and audit rows.
- Public form creates one valid report and cannot read/edit admin data; duplicate and invalid submissions are rejected; notifications do not affect persistence.
- Admin routes and MCP reject missing/invalid JWT, wrong issuer/audience, or non-allowlisted email. Access policy and Worker allowlist match.
- Product edits preserve history; order quantities are always user-entered; no accidental unit conversions or cross-unit summation.
- Daily/month/year/range sales/cost summaries explain estimate completeness and actual-sales override; stock and orders are separately visible.
- Mobile forms and lists are usable without clipped labels or uncontrolled horizontal overflow.
- CSV, order text sharing, and A4 portrait multi-page PDF preview/save work.
- MCP exposes only validated, audited tools and no direct database query surface.
- Build/typecheck and migration/API/auth checks pass. Deploy only on request and confirm the requested domain after deployment.
