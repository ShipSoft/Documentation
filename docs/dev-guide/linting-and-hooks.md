# Linting & git hooks

All active SHiP pixi repositories share one linting setup, run through
[`prek`](https://github.com/j178/prek) — a drop-in replacement for
`pre-commit`. This page covers how to install and run it locally; the
same hooks run in CI. sea-cucumber is the one partial exception, see
[below](#sea-cucumber).

## Why prek (and not `pre-commit`)

The hook *tools* (`ruff`, `clang-format`, `codespell`, `reuse`,
`gersemi`, …) come from a dedicated pixi `lint` environment, so their
versions are pinned in `pixi.lock` and resolved from `conda-forge` /
[`prefix.dev/ship-experiment/stable`](https://prefix.dev/channels/@ship-experiment/stable). That means:

- **No per-hook toolchain downloads** — prek does not fetch and build
  its own copies of the tools; it uses the ones pixi already provides.
- **Identical results everywhere** — the same pinned versions run on
  your machine, on a collaborator's, and in CI (including on NixOS,
  where `pre-commit`'s bundled binaries often fail).
- **One place to bump versions** — update the `lint` environment in
  `pixi.toml` / `pixi.lock`, not a separate `.pre-commit-config.yaml`
  toolchain.

## Install the hooks (once per clone)

From any repository checkout:

```bash
pixi run install-hooks
```

This installs both the `pre-commit` and `commit-msg` git hooks and
rewires them to run through `pixi run -e lint`, so the pinned toolchain
is always used — even when you commit from an editor or GUI.

!!! note "Coming from `pre-commit`?"
    Do **not** run `pre-commit install`. Use `pixi run install-hooks`
    instead; running `pre-commit` directly would use whatever tool
    versions happen to be on your `PATH` rather than the pinned ones,
    and may not work at all on some platforms.

## Commit messages

The `commit-msg` hook runs [commitizen](https://commitizen-tools.github.io/commitizen/)
(`cz check`), which rejects any message that isn't a
[Conventional Commit](https://www.conventionalcommits.org/), for
example:

```text
feat(output): publish EventHeader from every generator source
fix: build the SHiP siblings from source in the dev environment
docs(dev-guide): document the nightly dev builds
```

The same check runs on every commit of a pull request in CI (see
below), so installing the hook just gets you the error before you
push. Merge and revert commits are exempt. The convention matters
beyond style: release notes and changelogs are generated from the
commit messages with [git-cliff](https://git-cliff.org/) (see
[CI & repository conventions](ci-and-conventions.md#releases)).

## Run the hooks manually

To check the whole tree at any time (the same thing CI does):

```bash
pixi run lint
```

This is equivalent to `prek run --all-files`. To run against only
staged files, commit as usual — the installed hook runs automatically.

## clang-tidy

The C++ repositories also run [clang-tidy](https://clang.llvm.org/extra/clang-tidy/)
against a shared `.clang-tidy`. It isn't a prek hook, because it needs
a compilation database from a configured build. Most of them provide
two tasks:

```bash
pixi run clang-tidy        # whole tree
pixi run clang-tidy-diff   # only lines you changed relative to origin/main
```

`clang-tidy-diff` approximates what CI enforces, but it only checks
files that are in the compilation database. A change to a header alone
isn't analysed locally; run the full `clang-tidy` task or let CI report
it.

## In CI

Each repository's `Lint` workflow calls two reusable workflows from
[`ShipSoft/.github`](https://github.com/ShipSoft/.github):

- `prek.yml` runs the hooks. It is check-only and fails on any diff,
  and when the caller grants `pull-requests: write` it posts the hook
  output as a single sticky PR comment.
- `commit-check.yml` runs `cz check` over every commit in the pull
  request. It only runs on `pull_request` events, so it's skipped in
  the merge queue.

A final `Lint passed` job aggregates the two, and that job is the one
the branch ruleset requires. Because CI and your machine draw the hook
tools from the same pinned `lint` environment, a green `pixi run lint`
locally means `prek.yml` will pass on the PR. It doesn't cover commit
messages: the `commit-msg` hook checks those as you commit, and
`commit-check.yml` checks them again on the PR.

C++ repositories also call `clang-tidy.yml` from a separate workflow.
It is a ratchet: on a pull request it fails only on findings in lines
the patch adds or changes, while on a push to `main` it reports the
whole tree without failing. A changed header is checked through the
translation units that include it.

The shared configuration files (`AI_POLICY.md`, and for C++ repos
`.clang-tidy` and `cliff.toml`) are not edited per repository. A
monthly `config-sync.yml` job opens a PR that copies them from
`ShipSoft/.github/sync/`, so change them there. See
[CI & repository conventions](ci-and-conventions.md) for the full
list of shared workflows.

## sea-cucumber

sea-cucumber uses prek and a `lint` environment like the others, but
differs in three ways:

- its `install-hooks` task doesn't rewire the hooks to
  `pixi run -e lint`, so the hook tools may not be found unless you
  commit from inside `pixi shell -e lint`;
- it has no commitizen hook and doesn't call `commit-check.yml`, so
  nothing checks commit messages there (follow the convention anyway);
- lint runs as a step of its single `ci.yml` job (`pixi run lint`)
  instead of through `prek.yml`.

## Troubleshooting

- **A hook reformats a file and the commit "fails".** That is expected:
  prek applies the fix, aborts the commit, and leaves the change staged.
  Review it and commit again.
- **The `commit-msg` hook rejects your message.** Rewrite it as
  `type(scope): summary` with a type such as `feat`, `fix`, `docs`,
  `build`, `ci`, `chore`, `refactor` or `test`; `git commit --amend`
  fixes the last one.
- **`pixi run install-hooks` / `pixi run lint` is missing.** The repo may
  predate the migration or not define a `lint` environment; check
  `pixi.toml` for a `[feature.lint]` section and the `install-hooks` /
  `lint` tasks.
- See also the general [Troubleshooting](troubleshooting.md) page.
