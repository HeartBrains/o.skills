# Operations, deployment, and troubleshooting

## Configuration inventory

Use names and placeholders only in committed documentation. Typical non-secret configuration may include a source spreadsheet ID, worksheet name, API base URL, allowed frontend origins, cache TTL, and environment name. Treat IDs as project-specific configuration and never copy them from a reference deployment.

Secrets are needed only for private source access, LLM/MCP write actions, or deployment automation. Names vary by implementation; use secret storage for Google OAuth client credentials/refresh token, LLM provider key, and Cloudflare deployment token. Never provide example values that resemble real credentials.

## Access patterns

- **Public blog:** share the Sheet and Docs as viewer only when the content is intended for public use; test GViz CSV and Doc export without a signed-in session. Disable access to drafts by separating them into a private source or filtering before publication.
- **Private editorial source:** use OAuth/API permissions on the server. The client receives only the filtered public API response. Verify token refresh, revocation, and error handling.
- **Cloudflare:** create a Worker with project-specific name, variables, routes, and observability. Bind only required services. For D1/R2 use the appropriate binding; do not send credentials to client assets.
- **Frontend:** build and deploy to Pages or Worker assets according to the existing repository. Keep production URLs configurable; do not hardcode another project's host or IDs.

## CI/CD decision

CI/CD is not required when the Worker fetches public content at request time; a Sheet/Doc edit becomes visible after the edge cache expires or is purged. CI/CD is appropriate for code deployments and required for a build-time/static content sync. A scheduled Worker/job may sync content without rebuilding the frontend, but must validate and publish atomically.

For deployment automation, run schema/content validation and build checks first, then deploy with least-privilege Cloudflare credentials from a CI secret store. Avoid printing environment variables. Use preview deployments for frontend changes and a deliberate production release step.

## Validation checklist

- Test an anonymous fetch of the CSV and exported Doc if public delivery is intended; verify status, content type, and expected headers.
- Test quoted commas, escaped quote, newline, BOM, missing headers, duplicate slug/ID, invalid date, draft row, malformed Google Doc URL, empty Doc, oversized response, and Google errors.
- Confirm the detail body is sanitized, that no script/event handler survives, and that link/image URLs resolve on the public domain.
- Change a test title/body/status and verify expected cache refresh behavior, including bypass or purge.
- Verify no unpublished row or internal column appears in API output.
- Exercise the real frontend route in an anonymous browser and inspect console/network failures. Browser-visible images are the definitive image check.
- Review the built output, repository diff, and logs for credential values or private content.

## Common failures

| Symptom | Likely cause | Action |
| --- | --- | --- |
| Sheet endpoint returns HTML or sign-in page | Sheet is not readable anonymously, wrong tab name, or URL is not the CSV export | Confirm intended public sharing, exact tab, and CSV response; use OAuth adapter for private source. |
| Blog list works, article body is empty | Doc URL/ID malformed, Doc not readable by Worker, export failed, or sanitizer removed unsupported content | Check safe status/error, anonymous export access, exported HTML, and sanitizer allowlist. |
| Text appears but Doc images are broken | Export uses relative/temporary/permission-bound URLs or sanitizer rejects them | Use stable public HTTPS image URLs or copy approved assets to controlled storage; test signed-out. |
| Changes are delayed | Edge/API cache or frontend framework cache | Document TTL; bypass/purge the right cache layer and verify canonical cache keys. |
| Some rows are missing | Status filter, invalid required columns, malformed CSV, duplicate slug policy | Return row diagnostics in logs/admin tooling without exposing private row values publicly. |
| Site fails during Google outage | Runtime-fetch dependency | Add a previously validated stale cache or move to scheduled/build-time sync if availability requires it. |
