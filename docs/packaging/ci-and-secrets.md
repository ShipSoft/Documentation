# CI workflows and secrets

## Workflows in `ship-conda-recipes`

### `build.yml`: build and upload

Triggers on push to `main`, pull requests against `main`, the merge
queue (`merge_group`) and `workflow_dispatch`. The workflow itself is
a thin caller of the reusable
[`rattler-build-and-upload.yml`](https://github.com/ShipSoft/.github/blob/main/.github/workflows/rattler-build-and-upload.yml)
in `ShipSoft/.github`, which builds the whole channel with:

```bash
pixi run rattler-build build \
    --recipe-dir recipes/ \
    --channel https://prefix.dev/ship \
    --channel conda-forge \
    --skip-existing=all
```

rattler-build resolves cross-recipe dependency order from the
`requirements:` blocks; nothing else needs maintaining. On push to
`main`, each `output/{linux-64,noarch}/*.conda` file is uploaded with
`rattler-build upload prefix -c ship --skip-existing`.

PR builds run the same build step but **do not** upload. The packages
they produce are attached to the run as the `conda-packages` artifact
(kept for 7 days). An `All checks passed` job aggregates the result
for branch protection.

### `lint.yml`: hooks and commit messages

Runs the reusable `prek.yml` (all prek hooks, see
[Add a new recipe](add-a-recipe.md#linting)) and `commit-check.yml`
(Conventional Commits) on pushes, pull requests and the merge queue.

### `finalize-renovate-pr.yml`: refresh hashes on Renovate PRs

Runs on Renovate PRs that touch a `recipe.yaml` and pushes a commit
with the refreshed `source.sha256`. See
[Bump a package version](bump-a-version.md#renovate-managed-recipes).

### `channel-drift.yml`: weekly drift check

Mondays at 04:00 UTC. Compares the pins of every published package
against current conda-forge and opens up to two PRs (label `drift`):

- `channel-drift` bumps `build.number` on recipes whose pins have
  fallen behind; merging it rebuilds them.
- `variant-drift` advances the `root_base` pin in every
  `variants.yaml`. That is a real ROOT upgrade, so review it on its
  own.

Run it locally with `pixi run -e drift check-drift`.

### `channel-snapshot.yml`: weekly channel inventory

Mondays at 06:00 UTC. Fetches
`https://prefix.dev/ship/{linux-64,noarch}/repodata.json`,
renders `channel-snapshot.md`, and opens (or updates) a
`channel-snapshot` PR if the contents changed. The diff is the
changelog.

### Dev builds: early warning for downstream breakage

These jobs build downstream repositories from source against the
current channel, so breakage shows up before a recipe bump:

- `dev-build-fairship.yml` (daily, 05:00 UTC) and
  `dev-build-aegir.yml` (daily, 06:00 UTC) build the `main` branch of
  FairShip and aegir with a fresh dependency resolve (the committed
  `pixi.lock` is deleted). `workflow_dispatch` takes a `ref` input to
  build another branch.
- `dev-source-build-aegir.yml` (daily, 07:00 UTC) builds aegir's dev
  environment with data-model, Geometry and geometry_service as pixi
  source dependencies, as described in the
  [Developer Guide](../dev-guide/index.md). It pins pixi on purpose,
  to catch regressions in the preview pixi-build feature.
  `workflow_dispatch` takes `ref`, `data_model_ref`, `geometry_ref`
  and `geometry_service_ref` inputs to try out unmerged branches.

### `splines.yml`: GENIE splines

Manual (`workflow_dispatch`). Generates the production GENIE
cross-section splines in several stages, builds the
`genie-splines-ship` package from them and uploads it to the channel.
Inputs include the GENIE tune, the number of knots and the maximum
neutrino energy.

### Maintenance

- `update-lock-files.yml` calls the reusable `pixi-lock-update.yml`
  on the 1st of each month at 05:00 UTC and opens an `update-pixi` PR
  with the `pixi update` diff.
- `update-shared-configs.yml` calls the reusable `config-sync.yml` on
  the 1st of each month at 06:00 UTC and opens a PR when the files
  shared across ShipSoft repositories (`AI_POLICY.md`, `cliff.toml`)
  have changed.

## Secrets

- `PREFIX_DEV_API_KEY` — organisation secret. Used for
  `rattler-build upload prefix` by `build.yml` (on push to `main`)
  and `splines.yml`.
  Provisioned in ShipSoft GitHub org → Settings → Secrets and
  variables → Actions. Rotated from prefix.dev account settings.
- `GITHUB_TOKEN` — default per-run token. It covers the PRs opened by
  `update-lock-files.yml`, `channel-snapshot.yml` and
  `channel-drift.yml`, and the follow-up commit pushed by
  `finalize-renovate-pr.yml`. No extra configuration needed.

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
[`ShipSoft/.github`](https://github.com/ShipSoft/.github/tree/main/.github/workflows).
`ship-conda-recipes` uses `rattler-build-and-upload.yml`, `prek.yml`,
`commit-check.yml`, `pixi-lock-update.yml` and `config-sync.yml`. The
full list, and which repositories call each workflow, is in
[CI and repository conventions](../dev-guide/ci-and-conventions.md).

When adding a new ShipSoft repo, call these rather than inlining the
same setup-pixi boilerplate.
