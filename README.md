# webflow-embed-block

Code for the Tableau dashboard embed on the CTE Webflow site.

## Files

- **`webflow_loader.html`** — the small loader snippet that lives in Webflow's Page Settings > Custom Code on the dashboard page. Tracked here for reference; the Webflow paste is the runtime source of truth. Detects staging vs prod hostname and fetches the matching `embed_block.html` via jsdelivr.
- **`embed_block.html`** — the actual Tableau embed (Memberstack auth, JWT exchange, `tableau-viz` render). Served by jsdelivr to the loader at runtime.

## Branches and refs

- `main` — production embed. Cut releases by tagging (e.g. `v0.1.0`) and pointing the prod loader's `PROD_TAG` at the tag.
- `staging` — staging embed. Loader fetches `@staging` directly on Webflow staging sites and calls `jwt-generator-staging` Cloud Run.

## Production release flow

1. Iterate on `staging` branch. Webflow staging site picks it up immediately via `@staging`.
2. When ready, merge `staging` into `main` and tag the new release.
3. Bump `PROD_TAG` in **both** `webflow_loader.html` (this repo) and the Webflow Custom Code paste.
4. Optionally prime jsdelivr: `curl https://purge.jsdelivr.net/gh/Center-for-Transportation-Environment/webflow-embed-block@<new-tag>/embed_block.html`
