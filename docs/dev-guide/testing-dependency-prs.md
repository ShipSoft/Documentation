# Testing dependency PRs

## A PR of a SHiP package (no local checkout needed)

Source dependencies accept git refs, so a colleague's `data-model` PR
can be tested under `aegir` without checking anything out. In aegir's
uncommented dev block, use the git form with the PR branch (or an
exact commit):

```toml
[feature.dev.dependencies]
shipdatamodel = { git = "https://github.com/ShipSoft/data-model", branch = "their-feature" }
# or: { git = "...", rev = "<sha>" } to pin the exact commit
```

Then:

```console
$ pixi run -e dev build && pixi run -e dev smoke
```

If you have the PR checked out locally anyway, the `path` form does
the same and picks up further local edits.

As always, discard the manifest and lock changes afterwards:
`git checkout pixi.toml pixi.lock`.

## In CI, without a local setup

The [nightly dev builds](index.md#ci-coverage) in `ship-conda-recipes`
take refs as `workflow_dispatch` inputs, so you can run a PR through
them from the GitHub UI (Actions tab, "Run workflow") or with `gh`.
This needs write access to `ship-conda-recipes`.

`dev-source-build-aegir.yml` builds aegir's `dev` environment with
data-model, Geometry and geometry_service from source. Each sibling
defaults to its `main` branch; override any of them, and aegir itself,
with a branch, tag or SHA:

```console
$ gh workflow run dev-source-build-aegir.yml -R ShipSoft/ship-conda-recipes \
    -f data_model_ref=their-feature
```

The other inputs are `geometry_ref`, `geometry_service_ref` and `ref`
(aegir). The checkouts are always from the ShipSoft repositories, so
for a PR opened from a fork pass `refs/pull/<N>/head` instead of the
branch name. The job runs `pixi run -e dev install` and
`pixi run -e dev smoke` with a fresh lock.

`dev-build-aegir.yml` and `dev-build-fairship.yml` take a single `ref`
input for the aegir or FairShip revision and build it against the
released packages currently on the channels, again with a fresh lock.
That catches a PR that only works with the lock it was developed
against.

## An upstream phlex PR

phlex comes from conda-forge and has no pixi manifest, so it can't be
a source dependency. Instead, rebuild the conda package from the PR
source into the local channel:

```console
$ cd ship-conda-recipes
$ pixi run build-phlex-pr 687        # PR number, branch, tag or SHA
```

This resolves the PR head, builds
`recipes-dev/phlex/recipe.yaml` (a dev twin of the conda-forge
feedstock recipe) at that revision, and drops the package into
`../ship-local-channel` with build string `pr_<sha>` and build number
9999 — so it beats the released build both on channel priority and on
build number.

Consume it from aegir's dev environment (the commented block already
adds `../ship-local-channel` with priority; the `dev` and `phlex-pr`
flows compose):

```console
$ cd ../aegir            # dev block uncommented
$ pixi update phlex      # re-solve; picks pr_<sha> from the local channel
$ pixi list -e dev phlex # verify: build string pr_<sha>..., channel file://...
$ pixi run -e dev build && pixi run -e dev smoke
```

### Cleanup

```console
$ git checkout pixi.toml pixi.lock          # in aegir
$ rm -rf ../ship-local-channel              # optional: drop local builds
```

### Keeping the dev recipe honest

`recipes-dev/phlex/` must stay a diffable twin of the conda-forge
feedstock recipe — refresh it when the feedstock changes (see the
[dev-twin table](index.md#relationship-to-the-packaging-section)).
Its `variants.yaml` pins the ABI-relevant host packages (ROOT, spdlog,
fmt, python) to what aegir's lock currently uses; refresh those pins
when the stack moves.
