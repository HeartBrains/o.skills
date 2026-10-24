---
name: kyafq42026-headless-wordpress
description: Scaffold or maintain a headless WordPress project using JetEngine data and GraphQL on Hostinger Cloud hosting, with GitHub-triggered CI/CD. Use for the kyafQ42026 project or closely matching sites.
metadata:
  short-description: Headless WordPress, JetEngine, GraphQL, Hostinger CI/CD
---

# Headless WordPress + JetEngine + GraphQL on Hostinger

Use this skill to scaffold or maintain the project described as `kyafQ42026`: WordPress is the headless CMS on Hostinger Cloud Startup, JetEngine models dynamic content, GraphQL serves frontend data, and GitHub changes drive checks and deployments.

The `kyafQ42026` repository was not mounted in the environment where this skill was authored. When using the skill, inspect the real repository and Hostinger setup before changing code or deployment settings. If the repository or its deployment configuration is inaccessible, ask for its GitHub URL/access and continue only with repository-independent documentation until it is available. Do not invent the frontend framework, exact GraphQL plugin, existing JetEngine schema, branch, install path, or deployment credentials.

## Target architecture

- **CMS/backend:** WordPress installed and managed on the owner's Hostinger Cloud Startup site. WordPress database content, users, JetEngine configuration, plugin settings, media, and credentials remain on the hosting account; Git does not replace the CMS database.
- **Dynamic content:** JetEngine provides the site's actual Custom Post Types (CPTs), Custom Content Types (CCTs), custom fields, taxonomies, relations, and listings as currently configured. Preserve existing slugs, field keys, and relation IDs unless the owner approves a migration.
- **API:** Use the GraphQL plugin/add-on actually installed in the repository/site. Do not assume JetEngine CCTs, metadata, and relations appear in the GraphQL schema automatically. Verify each required type and field in staging; add/register the required schema support if missing.
- **Frontend:** First inspect the repository and keep its current frontend framework, build command, routes, and environment-variable naming. Do not replace it with Next.js, React, or another framework based only on this skill.
- **Hosting:** Hostinger Cloud Startup serves WordPress/PHP and, if the plan/account supports it, can connect a GitHub branch for automatic file deployment. Verify the current hPanel feature availability, selected branch, deployment history, PHP version, and SSH/SFTP access in the actual account.
- **Source control:** GitHub is the source of versioned custom code and the trigger for CI/CD. WordPress database content and uploaded media are backed up separately and are not committed to Git.

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
2. **Inventory the current system.** Record the frontend framework/build, WordPress and PHP versions, active GraphQL integration, JetEngine modules/CPTs/CCTs/fields/relations, public vs protected data, API base URL, staging environment, and current deploy mechanism. If a detail is unavailable, state the assumption and ask before changing that part.
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
