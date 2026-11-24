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

## Patching an upstream package in place

`build-phlex-pr` produces a proper package, but every change means a
full package build, and only phlex has a dev recipe. For a quick fix to
any CMake-built upstream package (phlex, ROOT, Geant4, ACTS, GeoModel,
…) there is a faster loop: build the source checkout incrementally with
the consumer's own pixi environment and install it over the released
package in that environment. Nothing downstream is rebuilt, which is
fine as long as your change keeps the ABI.

Two rules:

- **Check out the released version** of the package, the one
  `pixi list <package>` shows, and patch on top of it. Everything else
  in the environment was built against it.
- **Work in a throwaway worktree of the consumer.** The overlay changes
  the environment behind pixi's back: `pixi list` keeps showing the
  released build.

The `overlay` task in `ship-conda-recipes` does the bookkeeping. With
phlex under aegir:

```console
$ git -C aegir worktree add ../aegir.phlex-fix origin/main
$ git -C phlex worktree add ../phlex.fix v0.4.2   # the version in aegir's lock
$ (cd aegir.phlex-fix && pixi install)
$ cd ship-conda-recipes
$ pixi run overlay configure ../aegir.phlex-fix ../phlex.fix \
    -t "cetmodules pkg-config" -- \
    --preset default -DCMAKE_CXX_STANDARD=23 -DBUILD_TESTING=OFF \
    -DENABLE_CLANG_TIDY=OFF -DFORM_USE_RNTUPLE_STORAGE=ON
$ pixi run overlay build ../aegir.phlex-fix ../phlex.fix
$ pixi run overlay install ../aegir.phlex-fix ../phlex.fix phlex
$ (cd ../aegir.phlex-fix && pixi run build && pixi run smoke)
```

Then edit, `build`, `install` and rerun as often as you like. The CMake
arguments after `--` are the ones in the feedstock's `build.sh` (or the
`script:` section of its recipe). `-t` names build-only tools the
consumer environment lacks; they come from a cached `pixi exec`
environment. The build directory is `<src>-overlay-build` next to the
checkout. It belongs to the environment it was configured for: to
overlay the same checkout on another consumer, run `configure` again
for that one. To build with debug information, add
`-DCMAKE_BUILD_TYPE=Debug`.

For phlex under aegir, the first build takes about 4 minutes. After
that, a one-line change in `phlex/core` takes about 50 s to rebuild and
1.5 s to install, and aegir picks it up without being rebuilt. For ROOT
or Geant4 the first build takes as long as a full build of the package.

`pixi run overlay status ../aegir.phlex-fix` lists what is overlaid.
To undo:

```console
$ pixi run overlay restore ../aegir.phlex-fix phlex
```

This deletes every file that any `install` of the package put there,
including files a later build no longer installs, and has
`pixi reinstall` put the released package back.

If `install` fails part-way, the environment holds a mix of released
and patched files. The task records the overlay before it starts
installing, so `status` shows it with `state=installing` and `restore`
still puts the released package back. Files that the failed install
added and the package doesn't ship are not listed anywhere, though, so
they stay. If that matters, recreate the environment of the throwaway
worktree with `pixi clean` and `pixi install`.

### Why it installs into the environment

A separate prefix on `LD_LIBRARY_PATH` doesn't work: conda-forge
binaries are linked with `DT_RPATH` (`$ORIGIN/`), which the dynamic
loader searches before `LD_LIBRARY_PATH`, so they keep loading the
released libraries. Installing over them is safe for the package cache:
pixi hardlinks environment files from the shared cache, but
`cmake --install` replaces files rather than writing into them. Don't
copy files into the environment with `cp`: that would write through the
hardlink and change the package for every environment that uses it.
