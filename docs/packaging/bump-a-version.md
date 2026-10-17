# Bump a package version

Most recipes track upstream tags via
[Renovate](https://github.com/ShipSoft/ship-conda-recipes/blob/main/renovate.json).
Renovate runs at weekends and opens one PR per upstream release; review
the diff, merge if CI is green, and the new build flows to
`prefix.dev/ship-experiment/stable`.

## Renovate-managed recipes

`renovate.json` extends the shared ShipSoft preset
(`local>ShipSoft/.github`) and adds two `customManagers`, which read
`context.version` together with the source URL:

- GitHub sources: `url: https://github.com/<org>/<repo>/archive/…`
  (this includes `archive/refs/tags/`), looked up as GitHub tags.
- GitLab CERN sources: `url: https://gitlab.cern.ch/<group>/<project>/-/archive/…`,
  looked up as GitLab tags.

A leading `v` on the tag is stripped. If you copy a sibling recipe
([Add a new recipe](add-a-recipe.md)), its source URL already matches
one of these patterns and Renovate picks it up automatically.

When Renovate opens a PR, the `finalize-renovate-pr.yml` workflow runs
`ci/finalize-recipe-bump.sh` on the changed recipes. It downloads the
new tarball and pushes a follow-up commit with the refreshed
`source.sha256`. It skips recipes with more than one source URL (such
as `field-service`, which also fetches covfie); fix those hashes by
hand.

Still check that `build.number` was reset to `0`. Neither Renovate nor
the finalize step touches it.

## Releasing a SHiP package

SHiP's own packages (aegir, shipdatamodel, shipgeometry,
shipgeometryservice, field-service, shannon, trout, ship-random, …)
follow the same route:

1. Tag the release in the source repository (`v<X.Y.Z>`). Its
   `release.yml` calls the shared release workflow, which writes the
   release notes with git-cliff and creates the GitHub release.
2. At the next weekend run, Renovate opens the version bump in
   `ship-conda-recipes`.
3. Merge it once CI is green. The package is uploaded on push to
   `main`.

To get the package out sooner, open the bump PR yourself: edit
`context.version`, reset `build.number`, and update `source.sha256`.

## Dev twins and `[package].version`

Some artifacts outside `ship-conda-recipes` mirror release recipes for
the [developer workflow](../dev-guide/index.md) and must be bumped in
the same breath:

- the `[package].version` in the source repo's `pixi.toml`
  (data-model, Geometry, geometry_service): source-built dev packages
  must carry the released version or `run_exports` pins stop
  resolving;
- `field_service/recipe/recipe.yaml`, the dev twin of
  `recipes/field-service/recipe.yaml`;
- `ship-conda-recipes/recipes-dev/phlex/`, the dev twin of the
  conda-forge phlex feedstock recipe.

## Pinned recipes (manual)

Renovate is disabled in `renovate.json` for sources that don't follow
upstream release tags:

- `ShipSoft/pythia6` and `ShipSoft/ROOTEGPythia6`: ShipSoft forks
  built from fork tags (`context.tag`, e.g. `v6.4.28-ship1`).
- `webbjm/acts`: `acts-ship` is built from a commit
  (`archive/${{ commit }}.tar.gz`).
- `acts-project/covfie`: the second source of `field-service`, pinned
  to the tag that field_service's `CMakeLists.txt` fetches.

To bump one:

1. Edit `recipes/<pkg>/recipe.yaml`:
   - the `context` field the URL is built from (`tag`, `commit` or
     the hard-coded covfie version), and `context.version` if it
     changes. For `acts-ship`, a new commit always means a new
     date-based version, `0.0.0.dev<YYYYMMDD>+<sha7>`;
   - `source.sha256`: recompute.
2. Bump `build.number` if the version string stays the same but
   should produce a fresh package. Reset it to `0` whenever
   `context.version` changes.
3. Open a PR.

## ABI rebuilds across the channel

When conda-forge moves past a pin baked into a published SHiP package
(a new `root` or `geant4` build, for example), the weekly
[channel-drift job](ci-and-secrets.md#channel-driftyml-weekly-drift-check)
notices. It opens a `channel-drift` PR that bumps `build.number` on
every affected recipe. Moving the `root_base` pin itself comes as a
separate `variant-drift` PR. Merging either one rebuilds and
re-uploads the affected packages.

To rebuild by hand, for example for a change the drift check can't
see:

1. Bump `build.number` on every recipe that links the affected
   library.
2. Merge, or trigger `workflow_dispatch` on `build.yml`.
   `--skip-existing=all` builds the bumped build numbers as new
   packages and skips the rest.

For a smaller, targeted rebuild of a single recipe, use the
`recipe` and `force` inputs to `workflow_dispatch` (see
[CI workflows and secrets](ci-and-secrets.md)).
