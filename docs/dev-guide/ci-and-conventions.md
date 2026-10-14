# CI & repository conventions

The ShipSoft repositories share their CI, release tooling and
repository settings through
[`ShipSoft/.github`](https://github.com/ShipSoft/.github). Its README
is the reference for every workflow input; this page is the overview:
what exists, who uses it, and what it means when you open a pull
request.

## Reusable workflows

A repository calls these from its own `.github/workflows/` with
`uses: ShipSoft/.github/.github/workflows/<name>.yml@main`. Callers as
of 2026-10:

| Workflow | What it does | Called by |
| --- | --- | --- |
| `pixi-cmake-build.yml` | Runs a list of pixi tasks (`configure`, `build`, `test`, …) with `--locked`, compiling through ccache | aegir, aegir-genie, data-model, field_service, Geometry, geometry_service, Shannon, ship-ci-metrics, ship-random, Trout |
| `prek.yml` | Runs the prek hooks from the `lint` environment, check-only (see [Linting & git hooks](linting-and-hooks.md)) | every active pixi repository except sea-cucumber |
| `commit-check.yml` | Checks every commit of a PR against Conventional Commits with commitizen | same as `prek.yml` |
| `clang-tidy.yml` | clang-tidy ratchet: fails only on findings in changed lines | aegir, aegir-genie, data-model, field_service, Geometry, geometry_service, Shannon, ship-random |
| `pixi-lock-update.yml` | Scheduled PR that runs `pixi update` and shows the diff | every active pixi repository except sea-cucumber |
| `config-sync.yml` | Scheduled PR that copies the shared config files from `sync/` | all active repositories except sea-cucumber |
| `release.yml` | Publishes a GitHub release for a pushed `v*` tag, with notes from git-cliff | aegir, aegir-genie, data-model, field_service, Geometry, geometry_service, Shannon, ship-ci-metrics, ship-random, Trout |
| `doxygen-pages.yml` | Builds Doxygen and deploys it to GitHub Pages through Actions | aegir, aegir-genie, data-model, field_service, Geometry, geometry_service, Shannon, ship-random, Trout |
| `doxygen-gh-pages.yml` | The same, published to a `gh-pages` branch | FairShip |
| `physics-metrics.yml` | Stores physics metrics from `main` in git notes and compares PRs against them | aegir |
| `rattler-build-and-upload.yml` | Builds conda recipes and uploads them to `prefix.dev/ship` | ship-conda-recipes (see [CI and secrets](../packaging/ci-and-secrets.md)) |

The workflows are referenced by `@main`, so a change in
`ShipSoft/.github` reaches every caller on its next run.

## What a pull request has to pass

Every repository listed in
[`repo-config/repos.json`](https://github.com/ShipSoft/.github/blob/main/repo-config/repos.json)
gets the same two rulesets on its default branch:

- no deletion and no force-push, with no bypass for anyone;
- linear history, a merge queue, one approving review, resolved
  conversations, and squash or rebase as the only merge methods.
  Repository admins may bypass this one.

Instead of requiring individual jobs, the rulesets require two
aggregator jobs with fixed names: `All checks passed` in the build
workflow and `Lint passed` in the lint workflow. Each repository
defines them to fail if any other job in the same workflow failed or
was cancelled. Both workflows must also run on `merge_group`, or the
merge queue waits for a check that never arrives. `commit-check.yml`
is skipped in the queue, which counts as a pass: commit messages are
checked on the pull request.

Some repositories have a different tier in `repos.json`. sea-cucumber
builds, tests and lints in one job, so it only requires
`All checks passed`. Trout, htcondor_submission_scripts, tutorials and
`.github` itself have no required checks.

The settings are applied by hand with
`repo-config/apply-repo-config.sh` (a dry run prints the drift), not
by a scheduled job.

## Shared configuration files

Some files have to be identical everywhere. Their canonical copies
live in [`ShipSoft/.github/sync/`](https://github.com/ShipSoft/.github/tree/main/sync),
and each repository's `config-sync.yml` caller opens a PR on the first
of the month when its copy has drifted:

- `AI_POLICY.md` in every repository;
- `.clang-tidy` and `cliff.toml` in the C++ repositories.

Edit these in `ShipSoft/.github`, not in the repository you're working
on; a local edit is overwritten by the next sync. A repository that
needs extra clang-tidy settings adds a `.clang-tidy` with
`InheritParentConfig: true` in a subdirectory instead. `.clang-format`
is deliberately per repository.

## Naming

`SHiP` is the only top-level C++ namespace. Everything nested in it is
`lower_case` (`SHiP::geometry`, `SHiP::aegir::g4`), and a type inside
it doesn't repeat the prefix (`SHiP::GeometryService`, not
`SHiP::SHiPGeometryService`). The shared `.clang-tidy` enforces the
case rule. A `forbid-ship-namespace` hook that rejects the old
lower-case `ship` namespace exists in `ShipSoft/.github`, but can't be
used until that repository has its first tag.

## Dependency updates

Two mechanisms keep dependencies current:

- `pixi-lock-update.yml` refreshes `pixi.lock` on the first of each
  month, within the version ranges in `pixi.toml`.
- [Renovate](https://docs.renovatebot.com/) bumps the ranges
  themselves, GitHub Actions versions and pre-commit revisions.
  Repositories opt in with a `renovate.json` that extends the shared
  preset, `local>ShipSoft/.github`. It runs at weekends with at most
  five open PRs and compares conda packages with conda versioning.

## Releases

data-model, Geometry, geometry_service, field_service and Trout
release with a `scripts/release.sh` in the repository:

1. `scripts/release.sh X.Y.Z` bumps the version in `CMakeLists.txt`
   (and, where present, `CITATION.cff`, the `[package]` version in
   `pixi.toml` or the dev recipe), regenerates `CHANGELOG.md` with
   git-cliff and creates a
   `chore(release): vX.Y.Z` commit.
2. Open a pull request with that commit and merge it.
3. Tag the release commit on `main` as `vX.Y.Z` and push the tag.
4. `release.yml` publishes the GitHub release with notes generated
   from the Conventional Commits since the previous tag.

The script's `--help` lists repository-specific steps (data-model, for
example, also writes a frozen compatibility reference file). Getting
the new version onto `prefix.dev/ship` is a separate step in
`ship-conda-recipes`; see [Bump a version](../packaging/bump-a-version.md).

## Physics metrics

aegir compares a set of physics metrics between a pull request and
`main` on every PR. The tools come from
[ship-ci-metrics](https://github.com/ShipSoft/ship-ci-metrics)
(`ship-metrics-extract`, `-compare`, `-render`, `-pr-comment`), which
started as FairShip's CI scripts. The CI job extracts metrics from its
output files according to a `metrics_config.yaml`, and
`physics-metrics.yml` stores them in git notes on `main` and posts a
comparison comment on pull requests. Each metric declares whether it
must match exactly, within a relative tolerance, or within its
statistical uncertainty.

## Shared random numbers

aegir, aegir-genie, Shannon and Trout take their random number
generator from [ship-random](https://github.com/ShipSoft/ship-random),
a header-only Philox generator (`SHiP::random::PhiloxRng`) seeded per
event from a seed, a stream key and a counter. Results are therefore
reproducible regardless of the number of threads. Use a different
stream key for each independent purpose.
