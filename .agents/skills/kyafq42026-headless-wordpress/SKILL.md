---
name: kyafq42026-headless-wordpress
description: Scaffold or maintain a headless WordPress project using JetEngine data and GraphQL on Hostinger Cloud hosting, with GitHub-triggered CI/CD. Use for the kyafQ42026 project or closely matching sites.
metadata:
  short-description: Headless WordPress, JetEngine, GraphQL, Hostinger CI/CD
---

# Headless WordPress + JetEngine + GraphQL on Hostinger

Use this skill to scaffold or maintain `kyafQ42026`: WordPress is the headless CMS on Hostinger Cloud Startup, JetEngine models dynamic content, and GitHub changes drive checks and deployments. The intended setup includes GraphQL, but the currently checked-in frontend uses the WordPress REST API; verify the real implementation before assuming GraphQL is active or migrating APIs.

The `kyafQ42026` repository was not mounted in the environment where this skill was authored. When using the skill, inspect the real repository and Hostinger setup before changing code or deployment settings. If the repository or its deployment configuration is inaccessible, ask for its GitHub URL/access and continue only with repository-independent documentation until it is available. Do not invent the frontend framework, exact GraphQL plugin, existing JetEngine schema, branch, install path, or deployment credentials.

## Target architecture

- **CMS/backend:** WordPress installed and managed on the owner's Hostinger Cloud Startup site. WordPress database content, users, JetEngine configuration, plugin settings, media, and credentials remain on the hosting account; Git does not replace the CMS database.
- **Dynamic content:** JetEngine provides the site's actual Custom Post Types (CPTs), Custom Content Types (CCTs), custom fields, taxonomies, relations, and listings as currently configured. Preserve existing slugs, field keys, and relation IDs unless the owner approves a migration.
- **API:** Use the API the frontend currently consumes. If GraphQL is the approved target, identify the installed GraphQL plugin/add-on and verify JetEngine CCTs, metadata, and relations in its schema; do not assume they are exposed automatically. The current `khaoyaiart-next` frontend uses WordPress REST, so a GraphQL migration is a separate, explicit architecture change.
- **Frontend:** First inspect the repository and keep its current frontend framework, build command, routes, and environment-variable naming. Do not replace it with Next.js, React, or another framework based only on this skill.
- **Hosting:** Hostinger Cloud Startup serves WordPress/PHP and, if the plan/account supports it, can connect a GitHub branch for automatic file deployment. Verify the current hPanel feature availability, selected branch, deployment history, PHP version, and SSH/SFTP access in the actual account.
- **Source control:** GitHub is the source of versioned custom code and the trigger for CI/CD. WordPress database content and uploaded media are backed up separately and are not committed to Git.

## Two-site headless frontend, HTML, and rendering modes

When working in `HeartBrains/khaoyaiart-next`, use this verified architecture as the starting point, then recheck the current source because implementation details can change:

- The frontend serves two editorial brands from one frontend project and one WordPress API at `content.khaoyaiart.org`. `/bk/` is the public prefix for the internal site key `bkkk`; `/kyaf/` maps to `kyaf`. Site-specific components live under `components/bkkk/` and `components/kyaf/`, while shared API, mapping, and UI code lives under `lib/`, `utils/`, and shared component folders.
- This is a **logical two-site content model**, not evidence that WordPress Multisite (a network of separate WordPress sites) is enabled. The frontend currently reads the WordPress REST API (`/wp-json/wp/v2`), and filters records by the `meta.site` value (`bkkk` or `kyaf`); records without a site value are treated as shared in list fetching. Do not assume GraphQL is the live frontend data path just because JetEngine or a GraphQL plugin is installed. Inspect the active API calls and the deployed WordPress schema before changing either.
- Keep the owning site key with each related record. Construct links from the target record's own site, content type, and slug: use `/bk/<type>/<slug>/` for `bkkk` and `/kyaf/<type>/<slug>/` for `kyaf`. Do not derive a related record's destination from the page currently being viewed. Preserve the one-way relation behavior unless the owner requests reciprocal relations.
- Detail routes are implemented under both site prefixes and use separate site-specific page/detail components. For new route prefixes or content types, update the route matching and detail shell in `app/not-found.tsx` as well as the corresponding route page and component; verify the actual route map before relying on this instruction.

### Cross-site relationship fields

In the current `kyaf-catalog-schema` plugin, cross-site related records use custom post meta and an editor meta box; do not mistake this implementation for a JetEngine native relation. Verify the active plugin and field keys before changing or migrating the data model.

| Field | Stored form | Purpose |
| --- | --- | --- |
| `related_content_ids` | Ordered array of integer WordPress post IDs | Editable source of truth. The picker saves IDs in display order; sanitization removes invalid/duplicate IDs. |
| `related_content_json` | JSON string containing an ordered array of normalized records | Frontend payload regenerated from the selected IDs when the source is saved and when a related target changes. Registered as REST-visible post meta. |
| `site` | Existing record meta value: `bkkk` or `kyaf` | Identifies each target record's owning site and determines its public route. It is not a relationship direction or a display label. |

Each normalized `related_content_json` item currently contains `id`, `slug`, `type`, bilingual `title` (`en` and `th`), `date`, `image`, `category`, `status`, and `site`. The route type values are `exhibitions`, `activities`, `moving-image`, `artists`, and `blog`; source WordPress post types map to these public route names. `moving-image` currently has a detail route only on BK (`bkkk`). Keep the payload and frontend type allowlist in sync when a route is added or renamed.

The editor picker searches published exhibition, activity, moving-image, artist/residency, and blog records across both site keys. The save path must reject drafts/unpublished targets, unsupported post types or routes, invalid site keys, duplicates, and self-links. Relations are one-way: storing A → B must not silently create B → A. Preserve ordering and allow any number of valid targets unless the owner sets a product limit.

The frontend reads `meta.related_content_json`, parses and validates `site` and `type`, then builds the link from that item's own site and slug. Related cards show the title with the date beneath it and do not display a site-name label; keep the site key in the data for routing. Check an unauthenticated REST response for the relation payload and only expose the fields needed by the public frontend. A meta registration's `show_in_rest` flag alone does not prove the intended public request can read the value when a capability callback is present.

### Rendering WordPress HTML

- Prefer the existing `utils/richContent.tsx` `RichContent` component where its behavior fits. It decodes a small set of HTML entities, detects HTML, adds links to bare URLs, and renders markup with `dangerouslySetInnerHTML`; plain text is split into paragraphs. It is a client component and **does not sanitize HTML**.
- Treat CMS HTML and editor-provided CSS as input that needs a defined trust boundary. Before adding or reusing raw HTML/CSS injection, verify who can edit the value and use an established sanitizer/allowlist appropriate to the content. Do not claim the existing helper is safe or sanitized, and do not add untrusted markup directly to the DOM. Preserve needed semantic formatting, links, and language variants when sanitizing.
- Decide whether page content must exist in the initial HTML response. Content loaded by a browser `useEffect` appears only after hydration and is not part of the generated detail HTML. For initial HTML, fetch the public content in a server component during build (SSG) or on the request (SSR), then pass it to the renderer; keep private credentials server-side.

### SSG and SSR behavior

- The checked-in frontend currently sets `output: "export"` and `trailingSlash: true` in `next.config.ts`. Next's static export emits route HTML/assets under `out/`; there is no Next.js request-time server in that deployment, so it cannot perform SSR after a visitor request.
- Detail route pages use `generateStaticParams()` to fetch WordPress slugs during the build. Several detail pages also set `dynamicParams = false`. The page component can fetch data at build time for metadata or JSON-LD, while the visible detail client component may separately fetch the record in the browser through `useWPData`. Inspect both layers; do not call a page fully server-rendered just because its route has a server component or static HTML shell.
- New or changed WordPress records are not automatically present in the exported HTML. A build/deploy is needed to regenerate static routes unless the relevant static-host fallback renders that route client-side. `app/not-found.tsx` has a client-side detail shell for selected prefixes/types; check its current `matchRoute()` and `DetailShell` coverage before promising runtime behavior. That shell is client rendering, not SSR.
- If a requirement needs request-time SSR, personalization, or fresh WordPress data on every request, first confirm a supported Next.js server runtime and deployment target. Read the repository's installed Next.js guides under `node_modules/next/dist/docs/`; then design the server deployment and change away from static export deliberately. Do not add SSR-only APIs while `output: "export"` remains enabled, and do not describe client fetching or a rebuild as SSR. If static hosting remains required, use build-time SSG or an explicitly designed client-side fetch instead.
- Confirm whether the production pipeline actually builds and publishes `out/`, which branch triggers it, and how new WordPress slugs reach the build. A static frontend build does not deploy WordPress PHP/plugin files.

## GitHub release and plugin handoff

- Before pushing, inspect the target repository's current workflow triggers and the changed-file paths. Push only through the repository's established branch/PR process, then check the actual GitHub Actions run and report its status.
- A successful frontend build does not install or deploy a WordPress plugin. If no workflow matches the changed paths, say so; do not trigger an unrelated workflow or claim that deployment occurred.
- For WordPress plugin edits, update the plugin source and matching distributable ZIP, check that the ZIP has the expected plugin directory structure, and push both through GitHub. Stop after the source/ZIP handoff: do not upload, install, activate, or test-install the plugin on WordPress, including staging, unless the user explicitly changes this boundary.
- For frontend edits, use the verified GitHub workflow and verify the requested public page after it completes. Do not edit or sync the KYAF application repository when the request is limited to this skill or another repository.

When the task uses `HeartBrains/khaoyaiart-next`, its current `deploy.yml` builds the static frontend on matching pushes to `master`, `wp_content_updated` repository dispatch, or manual workflow dispatch. Its push path filter excludes `wp-plugin/**`; a plugin-only push does not start that workflow automatically, and that workflow does not install plugins. Recheck the workflow before relying on these details.

## Protect the live WordPress installation during deploys

Hostinger Git deployment can overwrite files in its target directory, and its setup may require the selected install path to be empty. Never point a whole-repository deployment at a populated WordPress `public_html` directory unless the repository intentionally contains the complete deployable WordPress tree and the owner has approved that replacement.

For an existing live WordPress site:

- Prefer deploying only the version-controlled child theme, custom plugin, or frontend artifact to its exact target directory. Keep WordPress core, `wp-config.php`, `.htaccess`, uploads, cache, backups, and user content outside a broad sync/deploy operation.
- If using Hostinger's native Git integration, set a deliberate repository root and install path, verify what files the next deploy will replace, and use an isolated/empty target when required. Check the deployment history after each first deployment.
- If native deployment cannot target only the intended code safely, a separately authorized deployment task may use GitHub Actions after CI to transfer only the allowed paths over SFTP/SSH. Confirm SSH/SFTP is enabled for this Cloud plan and restrict the deployment user to the smallest practical directory. This is not part of the plugin source/ZIP handoff above.
- Do not run file deletion/synchronization flags against the site root without a reviewed file allowlist and a tested rollback. Never deploy database dumps, `.env`, `wp-config.php`, `wp-content/uploads`, cache, or backup archives from Git.
- Before a separately authorized plugin, theme, JetEngine schema, or migration installation, confirm backups of the WordPress database and files and use the approved staging process. For routine plugin edits under this skill's current handoff boundary, package and push only; do not install or activate.

Hostinger's Git integration and GitHub Actions are different layers. Choose one deployment mechanism and document it:

1. **Recommended simple flow:** GitHub Actions runs CI on pull requests; require successful checks to merge into a protected production branch. Hostinger's GitHub integration auto-deploys that branch after merge. Prevent direct pushes to the production branch so an unchecked commit cannot trigger Hostinger deployment.
2. **CI-gated action deploy:** Disable Hostinger auto-deployment. GitHub Actions runs checks and then deploys only the approved code paths via SFTP/SSH. Put host, user, port, private key, and target path in GitHub Actions secrets/variables; never place credentials in YAML, repository files, or logs.

Do not enable both deployment mechanisms for the same path unless the duplication is intentional. A push may otherwise deploy twice or allow Hostinger to deploy before a GitHub Actions gate finishes.

## GraphQL and data safety

- Identify the exact GraphQL provider and version (for example WPGraphQL, Gato GraphQL, or a site-specific implementation); inspect active plugins and the existing endpoint before adding or replacing anything.
- Verify representative queries for public posts, each JetEngine CPT/CCT, custom fields, taxonomy terms, relations, images, pagination, and filters in staging. JetEngine REST modules are separate capabilities; their existence does not prove equivalent GraphQL support.
- Keep private, draft, user, tenant, and admin-only fields out of unauthenticated GraphQL responses. Apply WordPress capability checks and GraphQL field/type authorization. A hidden UI field or obscure endpoint name is not access control.
- Limit exposed fields and query cost. Use the GraphQL provider's depth/complexity, persisted-query, rate-limit, and IDE/debug controls where available. Disable public debugging/IDE access in production unless the owner specifically needs it.
- If frontend queries must be authenticated, use a dedicated least-privilege WordPress account/token and keep it server-side. Never put an Application Password, cookie, or private token in browser JavaScript or a `NEXT_PUBLIC_*`/public build variable.
- Configure CORS to the actual frontend origin only where browser requests need it. CORS is not authentication and does not protect a public GraphQL schema.
- Treat GraphQL schema changes and JetEngine configuration changes as API changes: document breaking field/slug changes and update frontend queries in the same reviewed release.

## Scaffold workflow

1. **Inspect before editing.** Read repository instructions, project README, GitHub Actions workflows, package/composer files, WordPress theme/plugin structure, JetEngine exports or schema notes, GraphQL plugin/version, and deployment configuration. Do not read secret files or dump production data. Confirm the actual Hostinger target path and which branch auto-deploys.
2. **Inventory the current system.** Record the frontend framework/build, WordPress and PHP versions, active data API, JetEngine modules/CPTs/CCTs/fields/relations, public vs protected data, API base URL, staging environment, and current deploy mechanism. For a two-brand frontend, document whether the sites are a native WordPress Multisite network or logical site partitions, each site's route prefix/key, record ownership rules, and cross-site related-link format. Record which content is rendered into initial HTML, how CMS HTML/CSS is sanitized, and whether detail routes use SSG, SSR, or client-side fetching. If a detail is unavailable, state the assumption and ask before changing that part.
3. **Separate code from content.** Keep custom themes/plugins and frontend source in Git. Keep database records, JetEngine admin configuration, media uploads, and secrets in their intended systems. Commit exported JetEngine configuration only when the export/import workflow is verified and the export contains no private user data or credentials.
4. **Use real CI checks.** Use the repository's existing commands and inspect its path filters before pushing. Where this task calls for GitHub CI/CD, push through the repository's normal process and inspect the run result. Add PHP lint/static checks, Composer validation, JavaScript/TypeScript build checks, or GraphQL validation only when the project has the relevant tooling and the task includes CI setup; do not add fake checks that always pass. A push for paths excluded by workflow filters does not count as a CI run.
5. **Protect releases.** Use pull requests and branch protection for the production branch. Review the changed-file list and deployment allowlist. Back up staging/production data before database or JetEngine schema changes; prefer backward-compatible schema changes and staged rollout.
6. **Respect the requested release boundary.** For an explicitly requested frontend release, verify the selected Hostinger path and deploy only through the configured workflow. For plugin edits covered by the current handoff, package and push source plus ZIP, then stop before installing or activating on WordPress.
7. **Verify after deployment.** Check Hostinger deployment status/commit, HTTPS frontend, WordPress admin, GraphQL endpoint status and expected fields, media URLs, forms/previews, and critical pages. Confirm no private data leaked, no uploads/config files were overwritten, and rollback/backup remains available.

## Settings and secrets checklist

Use the names already expected by the repo/workflow where they exist; do not rename existing settings without updating all consumers. Store secrets only in Hostinger's environment/configuration or GitHub Actions secrets, as appropriate. Commit sanitized examples only.

### Hostinger and WordPress settings

- Hostinger Cloud Startup account/site, production domain, SSL, PHP version, WordPress installation path, database backup schedule, and staging site.
- Hostinger GitHub integration: authorized repository, deployment branch, root/install path, auto-deployment toggle, and deployment history; or SSH/SFTP access for a CI-driven path deploy.
- WordPress database credentials and WordPress authentication/salt values stay in Hostinger-managed `wp-config.php` or the site's secure settings. Do not include `wp-config.php` in Git.
- WordPress production URL and the frontend URL; set canonical URLs, preview/revalidation URLs, and CORS origins from the real deployment topology.
- JetEngine plugin/license and active modules are managed in WordPress. Record the required module list and schema export process, but never commit license keys or editor/user data.
- GraphQL endpoint URL and exact provider/version. Public endpoint URLs are generally configuration, not secrets; private query tokens are secrets and server-only.

### GitHub CI/CD variables and secrets

Use only entries required by the chosen deployment workflow:

| Example setting | Type | Use |
| --- | --- | --- |
| `WP_GRAPHQL_URL` | Variable | GraphQL endpoint used by build-time/server-side queries. |
| `WP_GRAPHQL_TOKEN` | Secret, optional | Least-privilege token only if private queries are required; never expose to the browser. |
| `HOSTINGER_SFTP_HOST` | Secret/variable | Hostinger SSH/SFTP hostname or IP for Actions deployment. |
| `HOSTINGER_SFTP_PORT` | Variable | Port shown in Hostinger's account settings; do not assume the same for every account. |
| `HOSTINGER_SFTP_USER` | Secret | Restricted deployment username. |
| `HOSTINGER_SSH_PRIVATE_KEY` | Secret | CI-only private key whose public half is installed in Hostinger; use a dedicated key and rotate if exposed. |
| `HOSTINGER_DEPLOY_PATH` | Variable | Exact permitted theme/plugin/build directory, not the whole WordPress root by default. |
| `HOSTINGER_KNOWN_HOSTS` | Secret/variable | Verified SSH host key material for strict host-key checking. |

If Hostinger's own GitHub integration deploys from a selected branch, configure its OAuth connection and branch in hPanel; do not invent or store a Hostinger password in GitHub Actions for that flow.

### Optional headless preview/revalidation

If the project has preview or cache-revalidation routes, use a fresh high-entropy shared secret (for example `HEADLESS_PREVIEW_SECRET`) stored on both sides in server-only settings. Validate it server-side, rate-limit the endpoint, keep it out of browser-delivered editor code when possible, and provide a rotation process. Do not assume the repository uses a particular plugin's token behavior; inspect it first.

No Google OAuth, Service Account, or Google API credentials are required by this scope. Add another provider only if the owner requests it and its purpose is documented.

## Acceptance criteria

- The skill is applied to the actual checked-out repository and records its frontend framework, GraphQL provider, JetEngine schema, production branch, Hostinger deployment method, and target path without guessing.
- GitHub CI runs meaningful project checks on pull requests. A protected production branch prevents unchecked direct pushes; merge to that branch triggers exactly one configured deployment path.
- Deployment affects only intended code/artifacts and never replaces the live WordPress database, uploads, `wp-config.php`, or unrelated `public_html` files.
- GraphQL exposes the intended JetEngine fields/relations with correct authorization; private records and fields are denied to public requests.
- WordPress/JetEngine data and uploaded media are backed up separately from Git. Staging and production do not share unsafe deployment or mutation paths.
- Secrets are only in Hostinger/GitHub secret stores, not source, build artifacts, browser code, logs, or documentation.
- Post-deploy checks verify the frontend, WordPress admin, GraphQL contract, and deployment commit; rollback instructions are documented.

## Current upstream references

Recheck these sources when using the skill because Hostinger plan features, plugin behavior, and supported versions can change:

- [Hostinger Git deployment](https://www.hostinger.com/support/1583302-how-to-deploy-a-git-repository/)
- [Hostinger SSH access for Web/Cloud plans](https://support.hostinger.com/en/articles/1583645-how-to-enable-ssh-access)
- [Hostinger SFTP support](https://www.hostinger.com/support/which-file-transfer-and-server-access-options-are-supported-at-hostinger/)
- [JetEngine Custom Content Types](https://crocoblock.com/knowledge-base/jetengine/how-to-create-custom-content-type/)
- [JetEngine REST API for CCTs](https://crocoblock.com/knowledge-base/jetengine/jetengine-how-to-display-custom-content-type-items-using-rest-api/)
- [WPGraphQL plugin](https://wordpress.org/plugins/wp-graphql/)
