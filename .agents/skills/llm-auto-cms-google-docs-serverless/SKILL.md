---
name: llm-auto-cms-google-docs-serverless
description: Design, build, or operate an LLM-assisted headless CMS that uses Google Sheets for structured content, Google Docs for long-form articles, and serverless APIs for delivery. Use for public content pipelines and controlled editorial automation; not for systems that require private Google data exposed without authorization.
metadata:
  short-description: Google Docs and Sheets headless CMS
---

# LLM Auto CMS: Google Sheets + Docs + Serverless

Use this skill when a Google Sheet is the editorial index, Google Docs hold article bodies, and a serverless service delivers content to a website. Treat the Sheet and Docs as the authoring source; the public website consumes a validated read model. An LLM may assist with drafting, metadata, validation, and authorized edits, but it must not publish or overwrite content without the user's requested workflow.

## Choose the delivery model

- **Runtime delivery:** a Worker reads the Sheet and Docs when an API cache misses. Choose this for simple operations and edits that should appear quickly; account for Google availability, sharing permissions, and cache expiry.
- **Build-time/static sync:** a trusted job fetches and validates content, then writes sanitized static data/assets for the frontend build. Choose this when SEO/static rendering or reliable site delivery independent of Google at request time matters. Use CI or a scheduled sync and keep the build credential in the CI secret manager.
- Keep a private Google source private. Public CSV/Doc export is only appropriate for content intentionally published to anyone with the URL. For private sources, use OAuth or a suitable Google API identity with least privilege and never expose its token to browsers.

## Workflow

1. Inspect the target repo and the actual Sheet/Doc layout; record the schema, owner workflow, publishing statuses, hosting, and whether public read access is intended.
2. Define stable columns, IDs, unique slugs, status values, date format, categories, tags, cover-image URLs, and the Doc URL/ID. Validate headers and every row; report malformed rows instead of silently publishing them.
3. Build a server-only adapter that parses CSV correctly (quoted commas, escaped quotes, and embedded newlines), normalizes values, filters to explicitly published rows, and returns a versioned JSON shape.
4. Fetch long-form Doc content by validated document ID. Extract body content, sanitize HTML at the server boundary, and keep only the tags and attributes the frontend needs.
5. Make cache behavior explicit: TTL, query-aware cache keys, bypass/invalidation path, stale-on-error policy, and response cache headers. Avoid caching errors or private responses publicly.
6. Map the API into typed frontend models, render empty/loading/error states, and test listing, filtering, article detail, mobile layout, metadata/SEO, links, and images.
7. If an LLM can write to Sheets/Docs, expose narrow tools (read row, draft/update metadata, create/update Doc, publish) with least-privilege OAuth, audit logging, validation, and explicit confirmation before publish/destructive edits. Do not expose arbitrary spreadsheet operations or raw credentials to the model.
8. Document source permissions, deployment steps, schema changes, cache refresh, rollback, and who owns editorial approval.

## Required safeguards

- Never put Google OAuth client secrets, refresh tokens, access tokens, service-account keys, Cloudflare credentials, or private sharing links in Git, prompts, generated skills, client bundles, logs, or screenshots. Use secret stores and sanitized examples containing names/placeholders only.
- Do not send unpublished or private Doc text to an LLM or third-party OCR/service unless the user has authorized that data flow.
- Treat Sheet cells and Doc HTML as untrusted input. Escape plain text; sanitize rich HTML server-side; reject unsafe URL schemes and event handlers; do not rely on frontend sanitization alone.
- Validate Google document IDs against expected URL forms and fixed Google hosts; never fetch arbitrary URLs supplied in a row. Add timeouts and response-size limits.
- Google Docs image rendering is not guaranteed by HTML export. Inspect the actual exported `img src`; relative, temporary, permission-bound, or Google-hosted URLs may fail for site visitors. Prefer a stable HTTPS CDN URL in the Sheet or copy approved media into a controlled public asset store such as R2. Preserve alt text and dimensions, and allow only safe HTTPS image sources.
- Keep published content separate from draft/private content. A public endpoint must never return drafts, private notes, editor identities, or hidden columns.
- Make failure visible: distinguish unavailable Sheet, invalid schema, missing/private Doc, and empty article body. Keep the website usable when optional content is missing; never silently replace a valid article with an empty page.

## References

- Read [architecture-and-schema.md](references/architecture-and-schema.md) for the content model, Worker API, cache choices, sanitization, and image handling.
- Read [llm-editorial-automation.md](references/llm-editorial-automation.md) when adding model-driven Sheet/Doc updates or MCP tools.
- Read [operations.md](references/operations.md) for permissions, deployment, CI, validation, and troubleshooting.

## Completion criteria

- A published Sheet row resolves to one stable article URL and a valid JSON response.
- Public listing filters never expose non-published rows; detail pages render sanitized Doc content.
- All visible images resolve in a real browser on the deployed site, with an intentional fallback for failures.
- Google permission, cache, API error, and malformed-row behavior are documented and observable.
- No credential value or private source data is present in code, docs, build artifacts, or logs.
