# webflow-embed-block

Code for the Tableau dashboard embed on the CTE Webflow site.

## Files

- **`webflow_loader.html`** — the small loader snippet that lives in Webflow's Page Settings > Custom Code on the dashboard page. Tracked here for reference; the Webflow paste is the runtime source of truth. Detects staging vs prod hostname and fetches the matching `embed_block.html` via jsdelivr.
- **`embed_block.html`** — the actual Tableau embed (Memberstack auth, JWT exchange, `tableau-viz` render). Served by jsdelivr to the loader at runtime.

## Branches and refs

- `main` — production embed. Tagged on each release (e.g. `v0.1.0`).
- `staging` — staging embed. Loader fetches `@staging` directly on Webflow staging sites and calls `jwt-generator-staging` Cloud Run.

## Versioning and Webflow

Tags on `main` are the versioning unit for production. The Webflow Custom Code paste references a specific tag (e.g. `v0.1.0`) via `PROD_TAG` in the loader. **Pushing a new commit to `main` does NOT update production** — production only moves when you cut a new tag AND bump `PROD_TAG` in the Webflow paste.

### Why tags, not @main

Pointing prod at `@main` would mean any push goes live to the dashboard within ~5 minutes (jsdelivr's branch cache TTL). Tags are immutable refs — a bad push to `main` can't propagate to prod. Releases become an explicit, auditable decision: tag, bump, publish.

### Cutting a prod release

1. Iterate on `staging` branch. Webflow staging picks it up via `@staging` within minutes of each push.
2. Once happy, merge `staging` into `main` (via PR, once branch protection is on).
3. Tag: `git tag v0.2.0 main && git push origin v0.2.0`.
4. Update `PROD_TAG` in **both** `webflow_loader.html` (this repo, then commit + push) AND in the Webflow Custom Code paste.
5. Publish the Webflow site (Designer → Publish → check `dashboard.cte.tv`).
6. Pre-warm jsdelivr so the first user request doesn't pay the cold-cache cost: `curl https://purge.jsdelivr.net/gh/Center-for-Transportation-Environment/webflow-embed-block@v0.2.0/embed_block.html`.

### Rolling back

Bump `PROD_TAG` in the Webflow paste back to the previous tag (e.g. `v0.1.0`) and republish. No GitHub change needed — old tags are still served by jsdelivr indefinitely.

### Staging

Staging tracks the `staging` branch directly with no tagging. Pushes to `staging` are live within ~5 minutes (jsdelivr branch cache TTL). If you need to see a change faster, purge: `curl https://purge.jsdelivr.net/gh/Center-for-Transportation-Environment/webflow-embed-block@staging/embed_block.html`.
