# Linting & git hooks

All active SHiP pixi repositories share one linting setup, run through
[`prek`](https://github.com/j178/prek) — a drop-in replacement for
`pre-commit`. This page covers how to install and run it locally; the
same hooks run in CI.

## Why prek (and not `pre-commit`)

The hook *tools* (`ruff`, `clang-format`, `codespell`, `reuse`,
`gersemi`, …) come from a dedicated pixi `lint` environment, so their
versions are pinned in `pixi.lock` and resolved from `conda-forge` /
[`prefix.dev/ship`](https://prefix.dev/channels/ship). That means:

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

## Run the hooks manually

To check the whole tree at any time (the same thing CI does):

```bash
pixi run lint
```

This is equivalent to `prek run --all-files`. To run against only
staged files, commit as usual — the installed hook runs automatically.

## In CI

Linting runs on every pull request via the shared reusable workflow
`prek.yml` in [`ShipSoft/.github`](https://github.com/ShipSoft/.github)
(see its `README.md`). It is check-only and fails on any diff, and — when
the caller grants `pull-requests: write` — posts the hook output as a
single sticky PR comment. Because CI and your machine draw the hook
tools from the same pinned `lint` environment, a green `pixi run lint`
locally means a green lint job on the PR.

## Troubleshooting

- **A hook reformats a file and the commit "fails".** That is expected:
  prek applies the fix, aborts the commit, and leaves the change staged.
  Review it and commit again.
- **`pixi run install-hooks` / `pixi run lint` is missing.** The repo may
  predate the migration or not define a `lint` environment; check
  `pixi.toml` for a `[feature.lint]` section and the `install-hooks` /
  `lint` tasks.
- See also the general [Troubleshooting](troubleshooting.md) page.
