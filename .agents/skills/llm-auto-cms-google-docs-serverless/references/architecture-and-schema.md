# Architecture and content schema

## Reference architecture

```text
Editors / approved LLM tools
       │ Google Sheets + Google Docs
       ▼
Server-only source adapter
       │ validate → normalize → sanitize → cache
       ▼
Cloudflare Worker JSON API ──────► React/Vite or other frontend
       │                                  │
       └─ optional static sync job         └─ typed rendering + SEO
          (CI or scheduler)
```

Keep Google access and parsing out of browser code. An API response should be a stable contract even if the Sheet layout changes. For a small public blog, a Worker may fetch public GViz CSV and public Doc HTML. For private data, use OAuth-backed Google APIs in the Worker or a controlled sync job. Never make a private sheet public just to simplify deployment.

## Suggested editorial columns

Use a unique `Post_ID` and `Slug`; keep names stable and explicitly document any renames. A typical set is:

| Column | Type / rule |
| --- | --- |
| `Post_ID` | Stable string ID; required. |
| `Title_TH` | Display title; plain text. |
| `Slug` | Unique URL-safe key; required. |
| `Category` | Controlled category label. |
| `Published_Date` | ISO `YYYY-MM-DD`; validate as a real date. |
| `Author` | Public display value only. |
| `Excerpt` | Plain text summary; escape on render. |
| `Cover_Image_Url` | Stable HTTPS image URL. |
| `Tags` | Comma-separated or a separately maintained relation; normalize to string array. |
| `Doc_URL` | Google Docs URL; parse the document ID, do not fetch arbitrary hostnames. |
| `Read_Time` | Display string or nonnegative number with a defined unit. |
| `Featured` | Boolean; parse only explicit true values as true. |
| `Status` | Controlled values such as `Draft`, `Review`, `Published`, `Archived`; publish only exact `Published`. |

Additional columns should be deliberately classified as public, internal, or private. Do not return the entire row blindly; map only approved public fields.

## Parsing and API contract

- Use an RFC 4180 compatible parser or a tested state machine that supports escaped quotes and newlines inside quoted cells. Trim headers, remove a UTF-8 BOM, and retain line/row numbers for diagnostics.
- Validate required headers once per source load. For each record validate unique ID/slug, date, status, field lengths, tag count, and URL format. Return a clear safe error for a bad schema; report/skips invalid rows according to a documented policy.
- Normalize tags to arrays and booleans to booleans. Use a response shape such as `{ success, count, total, data }`, where `data` contains only public fields.
- Common routes: `GET /api/blogs` with `category`, `tag`, `featured`, `limit`; `GET /api/blogs/:slug` returns metadata plus `content_html` or a structured content field. Encode route params and cap list limits.
- Return appropriate `Content-Type`, CORS only for intended site consumers, explicit status codes, and `Cache-Control`. Public endpoints should not disclose internal parser errors or source URLs unnecessarily.

## Article body and images

Google Docs export can produce HTML with markup and asset references that are specific to the export or document permissions. A sanitizer should:

- remove scripts, styles, forms, iframes, embedded objects, event handlers, IDs/classes/styles unless intentionally supported, and dangerous elements;
- allow only a small formatting set, for example paragraphs, headings, lists, emphasis, blockquotes, links, tables, and images;
- allow links only for safe `https:`, `mailto:`, or `tel:` targets as applicable; allow image sources only for HTTPS or a deliberately supported same-origin asset path;
- preserve `alt`, title, and safe dimensions; normalize relative source paths or remove them with a visible fallback;
- set frontend image sizing and lazy loading, and provide a broken-image fallback.

The most reliable authoring practice is to put stable public image URLs in a dedicated Sheet column or managed asset store and reference them from article content. Test images from an anonymous browser context after deploy. Do not assume an image visible to a signed-in editor will be visible to site visitors.

## Cache and rendering choices

Runtime cache: include the full canonical route and relevant filters in the key, omit cache-bypass flags from the key, use a modest TTL, cache successful public responses only, and expose an intentional refresh path. When a Sheet or Doc changes, cache expiry or purge determines when visitors see it. Consider a short stale fallback only for previously validated content.

Static sync: fetch, validate, sanitize, and write generated content as build artifacts; fail or clearly report invalid published rows. Do not commit generated private content. Use scheduled/manual triggers when Sheet changes cannot trigger the repository build. Preserve stable slugs and support rollback to a prior artifact.

For SPA frontends, ensure detail routes have valid browser routing fallback. For SEO-critical content, render or pre-render pages from the validated content model. Generate page title, description, canonical URL, and Open Graph images from public fields.
