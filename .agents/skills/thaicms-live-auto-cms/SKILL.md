---
name: thaicms-live-auto-cms
description: Maintain or extend ThaiCMS Live's headless blog pipeline using Google Sheets metadata, Google Docs article bodies, a Cloudflare Worker API, and the existing React/Vite frontend. Use for blog ingestion, rendering, cache, image, and deployment work in that repository.
metadata:
  short-description: ThaiCMS Google Docs blog pipeline
---

# ThaiCMS Live headless blog profile

For the general architecture, security, LLM editing, and troubleshooting rules, read [$llm-auto-cms-google-docs-serverless](../llm-auto-cms-google-docs-serverless/SKILL.md) and only the references relevant to the task.

## Project-specific architecture

- The site frontend is React + TypeScript + Vite with React Router; it is a static client application, not Next.js. Use the repository's current Vite build configuration and Cloudflare Pages/Worker setup rather than introducing a different framework.
- Blog metadata is fetched by the Cloudflare Worker from a Google Sheets GViz CSV endpoint. The Worker normalizes metadata, exposes published rows at `/api/blogs`, and supports category/tag/featured/limit filters.
- A blog detail route resolves the `Slug`, reads `Doc_URL`, fetches the Google Docs HTML export at the Worker, sanitizes it with `HTMLRewriter`, and returns `content_html`. The React detail page maps that field to its article body.
- The current delivery is runtime-fetch plus edge cache, with a five-minute response TTL and an explicit `nocache=true` bypass. This does not bake article content into the Vite build and does not require CI/CD for Sheet/Doc edits. Code changes still need the repository's established deploy workflow.
- Keep Google source configuration in Worker variables/project configuration, never in frontend code. Do not copy another environment's spreadsheet ID, domain, route, or Worker identifiers into a new project.

## Editorial and rendering invariants

- Filter to exact `Published` status before returning rows. Keep draft/internal columns out of the JSON response.
- Keep the existing public field contract and frontend mapper aligned. If changing columns or response fields, update the Worker parser, frontend types/mapping, list/detail behavior, and relevant documentation together.
- The Google Sheet and Docs must allow public viewer access for this public runtime-fetch design. If editors want private Docs, switch the server adapter to OAuth/API credentials in Worker secrets; never make a private source public just to fix a fetch.
- Doc image display is not automatic. Inspect the exported HTML URLs. The sanitizer accepts only safe URL forms; Google-relative or permission-bound sources can be stripped or fail for visitors. Put stable public HTTPS media URLs in the metadata/assets store or migrate approved images to controlled storage, then verify signed out in a browser.
- Keep HTML sanitization on the Worker before the frontend uses `dangerouslySetInnerHTML`. Do not weaken it to make one unsupported Doc element render.
- On a Doc permission/export error, return a visible API error and keep the detail page in a recoverable state; do not claim the article is static or available if it is fetched at runtime.

## Work sequence

1. Inspect `worker.js`, Wrangler config, `src/app/data/blogApi.ts`, blog list/detail components, and repo guidance before editing.
2. Verify the published Sheet tab headers, one representative public Doc export, and actual media URL forms without recording private content or credentials.
3. Make the smallest contract-preserving change. Validate CSV edge cases, status filtering, slug lookup, sanitizer output, and cache behavior.
4. Build with the repository's current Vite command and test the deployed/public routes only when authorized. Do not deploy or modify Google sharing settings unless requested.
5. Update this repo's docs when source schema, sharing, cache, or deployment behavior changes.

## Boundaries

- Never store OAuth credentials, refresh tokens, Cloudflare tokens, or real secret values in this skill or the repository.
- Do not use Figma publishing as the Cloudflare deployment mechanism. Use the current repository's Cloudflare Pages/Worker process.
- Do not create a Next.js replacement or add CI solely to refresh runtime-fetched content; choose build-time sync only when static content/SEO or Google-independent delivery is explicitly needed.
