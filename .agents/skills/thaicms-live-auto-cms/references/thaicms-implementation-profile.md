# ThaiCMS implementation profile

This profile records behavior observed in ThaiCMS Live and is intentionally free of its live spreadsheet ID, API domain, and Cloudflare resource identifiers.

## Worker contract

- `GET /api/blogs`: reads the configured GViz CSV, parses metadata, filters published rows, and supports `category`, `tag`, `featured=true`, and bounded `limit` filters.
- `GET /api/blogs/:slug`: finds a published row and fetches the Google Doc export HTML referenced by `Doc_URL`; responds with metadata plus sanitized `content_html`.
- `GET /health`/root service response can be used for basic availability, but it does not prove the Sheet or Docs are readable.
- CORS is enabled for public blog consumers. Successful responses use edge/browser caching; `nocache=true` bypasses cache lookup/storage for diagnosing source updates.

## Sheet contract

Required public columns: `Post_ID`, `Title_TH`, `Slug`, `Category`, `Published_Date`, `Author`, `Excerpt`, `Cover_Image_Url`, `Tags`, `Doc_URL`, `Read_Time`, `Featured`, `Status`.

`Tags` normalize to an array, `Featured` normalizes to boolean, and only `Status=Published` is exposed. Preserve stable IDs/slugs. Any schema change must be coordinated with the Worker parser and `blogApi.ts` mapper.

## Doc HTML and media

The Worker extracts `<body>` from Google Docs' HTML export, removes active/embedded elements, and removes non-allowlisted attributes. It retains safe HTTPS image URLs and strips unsafe or unparseable sources. Google Docs can return image references that are not stable public URLs; do not assume those will render. Prefer a stable CDN URL and test on the real public detail route while signed out.

The React detail component renders the sanitized API content as HTML. The Worker is the trust boundary. If more formatting is needed, extend the sanitizer with narrow tags/attributes and URL validation, then verify scripts and event attributes remain removed.

## Hosting and release

- The blog API is a standalone Cloudflare Worker, while the site is a React/Vite static frontend on Cloudflare Pages.
- Current frontend build uses the repository's configured Figma Vite config; inspect `package.json` and Wrangler/Pages setup at task time because project deployment settings can change.
- Blog content changes flow from Google at runtime and become visible after cache expiry/bypass. Frontend code changes require a Pages deployment. Worker code/config changes require a Worker deployment.
- Keep API base URL and Sheet configuration in appropriate environment/project configuration. Never copy live IDs or domains into a reusable scaffold.
