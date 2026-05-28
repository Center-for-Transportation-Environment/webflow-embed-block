# webflow-embed-block

Code for the Tableau dashboard embed on the CTE Webflow site.

## Files

- **`webflow_loader.html`** is the loader snippet that lives in Webflow's Page Settings > Custom Code on the dashboard page. Tracked here for reference. The Webflow paste is the runtime source of truth. Detects staging vs prod hostname, picks the right URL, and fetches the matching `embed_block.html`.
- **`embed_block.html`** is the actual Tableau embed (Memberstack auth, JWT exchange, `tableau-viz` render, viz sizing styles). Loaded at runtime by the loader. This is where you change embed dimensions and other render behavior.

## How content reaches the browser

```
Webflow page > loader (in Custom Code) > fetches embed_block.html > injects scripts > JWT exchange > renders Tableau
```

The loader picks the source based on hostname:
- Staging Webflow (`*.webflow.io`) uses `raw.githubusercontent.com/.../staging/embed_block.html`
- Prod Webflow (`dashboard.cte.tv`) uses `cdn.jsdelivr.net/.../@v0.1.2/embed_block.html`

DevTools console shows the active environment and version on every page load:
- `[STAGING] webflow-embed-block@staging` (green)
- `[PROD] webflow-embed-block@v0.1.2` (blue)

## Branches and refs

- `main` is the production embed. Protected. Changes go via PR. Tagged on each release (e.g. `v0.1.2`).
- `staging` is the staging embed. Push-open for fast iteration. Webflow staging site picks up pushes within about 5 min.

## Updating staging (anyone with push access)

```bash
git checkout staging
# edit embed_block.html
git add embed_block.html
git commit -m "..."
git push origin staging
```

Hard-refresh `zebra-sandbox.webflow.io/dashboard`. GitHub raw's cache means changes show up within about 5 min.

## Updating prod (cutting a release)

Prod does not auto-update from any branch. Releases are explicit, version-tagged commits on `main`.

1. Iterate on `staging` until happy.
2. Open a PR from `staging` into `main`. Get approval, or self-merge if solo.
3. After merge, bump `PROD_TAG` in `webflow_loader.html` (the in-repo copy) to the next semver. Commit on `main` via PR.
4. Tag the new commit: `git tag v0.2.0 main && git push origin v0.2.0`.
5. Update `PROD_TAG` in the Webflow Custom Code paste to match (one line change).
6. Publish Webflow to `dashboard.cte.tv`.
7. Optional: pre-warm jsdelivr with `curl https://purge.jsdelivr.net/gh/Center-for-Transportation-Environment/webflow-embed-block@v0.2.0/embed_block.html`.

## Rolling back

Change `PROD_TAG` in the Webflow paste back to a previous tag (e.g. `v0.1.0`) and republish. No GitHub change needed. Old tags are served by jsdelivr indefinitely.

## Why two different sources (jsdelivr vs GitHub raw)

Prod uses jsdelivr because tag URLs are cached immutably (1 year), so loads are fast and stable.

Staging uses `raw.githubusercontent.com` because jsdelivr's branch resolver never indexed this repo's branches (see Known issues). raw has a shorter cache (about 5 min) and serves branch HEADs directly with CORS enabled.

## Branch protection rules

The `main` branch is protected:
- Direct pushes blocked (PR required)
- Force pushes blocked
- Branch deletion blocked
- Linear history required (squash or rebase merges only)
- Admins can bypass the rule in emergencies

`staging` is unprotected and open for direct push.

## Known issues

### jsdelivr branch resolver

jsdelivr's GitHub branch indexer never populated this repo's branches (only its tags). We work around it by routing the staging path through `raw.githubusercontent.com`. Prod uses tag URLs, which work correctly.

### Never re-create a tag

If you tag `v0.2.0` and find a bug, cut `v0.2.1`. Do not force-move `v0.2.0` to a new commit. jsdelivr's cache makes recreated tags unreliable.
