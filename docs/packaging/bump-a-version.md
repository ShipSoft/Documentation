# Bump a package version

Most recipes track upstream tags via
[Renovate](https://github.com/ShipSoft/ship-conda-recipes/blob/main/renovate.json).
A weekend cron opens a PR per upstream release; review the diff,
merge if CI is green, and the new build flows to `prefix.dev/ship`.

## Renovate-managed recipes

The `customManagers` in `renovate.json` cover three patterns:

- GitHub tags via `archive/refs/tags/v<X>.tar.gz` URLs.
- GitHub tags via templated `context.version` + `archive/` URLs.
- GitLab CERN tags via templated `context.version` +
  `archive/` URLs.

If you copy a sibling recipe ([Add a new recipe](add-a-recipe.md))
its source URL will already match one of these patterns and Renovate
will pick it up automatically.

When Renovate opens a PR, check that:

- `build.number` was reset to `0`. Renovate doesn't reset it; you
  may need to edit the PR.
- `source.sha256` was bumped. Renovate doesn't refresh hashes for
  every pattern; if the PR fails to build with a hash mismatch,
  copy the expected SHA from the rattler-build error and push it.

## Branch-pinned recipes (manual)

A few recipes pin a branch or commit instead of a release tag.
These are disabled in `renovate.json`:

- `SHiP/FairShip`
- `ShipSoft/data-model`
- `ShipSoft/geometry_service`
- `SND-LHC/pythia6`
- `luketpickering/ROOTEGPythia6`

To bump one:

1. Edit `recipes/<pkg>/recipe.yaml`:
   - `context.version` — the new short SHA or branch name.
   - `source.url` — point at the new commit (`archive/<sha>.tar.gz`).
   - `source.sha256` — recompute.
2. Bump `build.number` if the upstream change doesn't change the
   version string but should produce a fresh package.
3. Open a PR.

## ABI rebuilds across the channel

When an ABI break lands in an upstream conda-forge package
(`root` major version, `geant4` major version, `libstdcxx`, etc.):

1. Bump `build.number` on every recipe that links the affected
   library. The current build numbers serve as the rebuild signal.
2. Trigger `workflow_dispatch` on `build.yml`. The
   `--skip-existing=all` flag will pick up the bumped build numbers
   as new packages to build and skip the rest.

For a smaller, targeted rebuild of a single recipe, use the
`recipe` and `force` inputs to `workflow_dispatch` (see
[CI workflows and secrets](ci-and-secrets.md)).
