# CI workflows and secrets

## Workflows in `ship-conda-recipes`

### `build.yml`: build and upload

Triggers on push to `main`, pull requests against `main`, the merge
queue (`merge_group`) and `workflow_dispatch`. The workflow itself is
a thin caller of the reusable
[`rattler-build-and-upload.yml`](https://github.com/ShipSoft/.github/blob/main/.github/workflows/rattler-build-and-upload.yml)
in `ShipSoft/.github`. It runs one native job per platform listed in
the `platforms` input (linux-64 on `ubuntu-latest`, linux-aarch64 on
an arm64 runner), and each job builds the whole channel with:

```bash
pixi run rattler-build build \
    --recipe-dir recipes/ \
    --channel https://prefix.dev/ship-experiment/stable \
    --channel conda-forge \
    --skip-existing=all
```

rattler-build resolves cross-recipe dependency order from the
`requirements:` blocks; nothing else needs maintaining. Recipes that
don't support a platform skip it (see [Platforms](index.md#platforms)).
On push to `main` and on manual dispatches from `main`, each job
uploads its `output/<platform>/*.conda` files, and the linux-64 job
also uploads `output/noarch/`, with
`rattler-build upload prefix -c ship-experiment/stable --skip-existing`.
The upload authenticates with trusted publishing (see
[Secrets](#secrets)).

PR builds run the same build step but **do not** upload. The packages
they produce are attached to the run as one
`conda-packages-<platform>` artifact per platform (kept for 7 days). An `All checks passed` job aggregates the result
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
`https://prefix.dev/ship-experiment/stable/{linux-64,linux-aarch64,noarch}/repodata.json`,
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

### `nightly-main.yml`: nightly builds of library `main` branches

Daily at 02:30 UTC. Builds the `main` branch of ship-random,
data-model, Geometry, geometry_service and field_service, and uploads
them to `prefix.dev/ship-experiment/nightly`. Each library builds
against released dependencies only (stable channel and conda-forge),
so a consumer can swap in exactly one `main` build. Consumer
repositories use this channel for advisory PR jobs through the
reusable `pixi-main-deps.yml`. Every library except the noarch
ship-random is built for both linux-64 and linux-aarch64.
`workflow_dispatch` takes `recipe` and `force` inputs.

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
  shared across ShipSoft repositories (by default only `AI_POLICY.md`)
  have changed.

## Secrets

- No prefix.dev API key. `build.yml`, `splines.yml` and
  `nightly-main.yml` upload with trusted publishing: the jobs grant
  `id-token: write`, and each workflow is registered as a trusted
  publisher on its channel (`@ship-experiment/stable` or
  `@ship-experiment/nightly`) in the prefix.dev channel settings. A
  new workflow that uploads needs its own entry there.
- `APP_CLIENT_ID` (variable) and `APP_PRIVATE_KEY` (secret) — the
  credentials of a GitHub App. `finalize-renovate-pr.yml` creates a
  short-lived app token from them and uses it to push the sha256
  refresh commit. Workflows don't start on pushes made with
  `GITHUB_TOKEN`, so the app token is what makes the build and lint
  checks run on that commit.
- `GITHUB_TOKEN` — default per-run token. It covers the PRs opened by
  `update-lock-files.yml`, `channel-snapshot.yml` and
  `channel-drift.yml`. No extra configuration needed.

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
