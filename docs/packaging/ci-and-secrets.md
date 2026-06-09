# CI workflows and secrets

## Workflows in `ship-conda-recipes`

### `build.yml` — build and upload

Triggers on push to `main`, pull requests against `main`, and
`workflow_dispatch`. The whole channel is built with one command:

```bash
pixi run rattler-build build \
    --recipe-dir recipes/ \
    --channel https://prefix.dev/ship \
    --channel conda-forge \
    --skip-existing=all
```

rattler-build resolves cross-recipe dependency order from the
`requirements:` blocks; nothing else needs maintaining. On push to
`main`, every `output/{linux-64,noarch}/*.conda` artifact is
uploaded with `rattler-build upload prefix -c ship --skip-existing`.

PR builds run the same build step but **do not** upload.

### `update-lock-files.yml` — monthly pixi lock refresh

Calls the reusable
[`ShipSoft/.github/.github/workflows/pixi-lock-update.yml@main`](https://github.com/ShipSoft/.github/blob/main/.github/workflows/pixi-lock-update.yml).
Runs on the 1st of each month at 05:00 UTC and opens an
`update-pixi` PR with the `pixi update` diff.

### `channel-snapshot.yml` — weekly channel inventory

Mondays at 06:00 UTC. Fetches
`https://prefix.dev/ship/{linux-64,noarch}/repodata.json`,
renders `channel-snapshot.md`, and opens (or updates) a
`channel-snapshot` PR if the contents changed. The diff is the
changelog.

## Secrets

- `PREFIX_DEV_API_KEY` — organisation secret. Used by
  `rattler-build upload prefix` in `build.yml` on push to `main`.
  Provisioned in ShipSoft GitHub org → Settings → Secrets and
  variables → Actions. Rotated from prefix.dev account settings.
- `GITHUB_TOKEN` — default per-run token; sufficient for the
  `peter-evans/create-pull-request` calls in
  `update-lock-files.yml` and `channel-snapshot.yml`. No extra
  configuration needed.

## Triggering a targeted rebuild

From the GitHub Actions UI, run `Build and upload conda packages`
with `workflow_dispatch`. Two inputs help when only one recipe
needs to be rebuilt (e.g. after Renovate bumps a single upstream
version):

- `recipe` — directory name under `recipes/`, e.g. `random123`.
  Restricts the build to that recipe; its dependencies are pulled
  from the channel.
- `force` — drops `--skip-existing` so a rebuild proceeds even
  when an artifact with the same build number is already on the
  channel. Pair with a `build.number` bump in the recipe.

## Reusable workflows

The cross-repo CI pieces live in
[`ShipSoft/.github`](https://github.com/ShipSoft/.github/tree/main/.github/workflows):

- `pixi-lock-update.yml` — monthly lock-file PR (used by
  `ship-conda-recipes`, `aegir`, `FairShip`).
- `pixi-cmake-build.yml` — generic pixi-driven CMake build with
  a JSON task array (used by `aegir`, `geometry`,
  `geometry_service`).
- `doxygen-gh-pages.yml` — Doxygen build + Pages deploy.

When adding a new ShipSoft repo, prefer calling one of these
rather than inlining the same setup-pixi boilerplate.
