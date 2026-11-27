# Coming from aliBuild

With aliBuild, `aliBuild init <package>` made any package in the stack a
development package: you edited the checkout, and the next
`aliBuild build` rebuilt it incrementally along with everything that
depended on it. pixi and conda-forge split that job differently, and
most of it is covered. This page maps the aliBuild habits onto the pixi
workflows and lists what is still missing.

Two things make the pixi side work with less rebuilding than aliBuild
did. Packages come prebuilt from conda-forge and
`prefix.dev/ship-experiment/stable`, and conda-forge keeps ABI stable
within a pinned build. A patch that doesn't change ABI therefore only
needs the package you're patching rebuilt, not its dependents.

## Workflow by workflow

| With aliBuild | With pixi | Notes |
| --- | --- | --- |
| `aliBuild init` a SHiP package (data-model, Geometry, geometry_service) and build a consumer against it | A path source dependency in the consumer's `dev` environment | [Multi-package development](multi-package.md). An edit reaches aegir in about 20–30 s. |
| The same for field_service | Build it into `../ship-local-channel` | Its multi-output recipe can't be a source dependency yet (gaps 1 and 5 in the [gap log](pixi-build-gaps.md)). |
| Build a colleague's branch of a SHiP package | A git source dependency, or the `dev-source-build-aegir.yml` workflow | [Testing dependency PRs](testing-dependency-prs.md) |
| Several development packages stacked on each other | Several path dependencies in one `dev` environment | Keep the released specs in a `[feature.released]` table (gap 10). |
| Dependents rebuilt automatically after a change | Only source dependents are rebuilt; binary dependents are reused | Fine while ABI holds, see the [ABI caution](multi-package.md#rules). |
| `aliBuild init` an upstream package (ROOT, Geant4, phlex, …) and patch it | `pixi run overlay` in `ship-conda-recipes`: build the checkout incrementally in the consumer's environment and install it over the released package | [Patching an upstream package in place](testing-dependency-prs.md#patching-an-upstream-package-in-place). The first build takes as long as the package's full build. |
| An upstream PR built as a proper package | `build-phlex-pr` into the local channel | phlex only, through a dev twin of the feedstock recipe. |
| A patch carried in `shipdist` | A patch in the feedstock (conda-forge) or in a `ship-conda-recipes` recipe | Goes through the normal [packaging](../packaging/index.md) route. |
| `--defaults` and version overrides for the whole stack | pixi environments and features, the `ship-nightly` channel | |
| `alienv enter` | `pixi shell`, `pixi run` | |
| Shared remote build store | prefix.dev channels | |

## What is still missing

- **Opting in is not local.** `aliBuild init` touched nothing that was
  committed. With pixi you uncomment a block in the committed
  `pixi.toml` and must not commit the result, because `--locked` checks
  every environment (gaps 2 and 3,
  [prefix-dev/pixi#6606](https://github.com/prefix-dev/pixi/issues/6606)).
- **Upstream packages have no package-shaped dev loop.** The overlay
  patches the environment directly. pixi doesn't know about it, so
  `pixi list` keeps showing the released build until you run
  `pixi run overlay status`. Use it on a throwaway worktree of the
  consumer, and restore before you rely on the environment for anything
  else.
- **ABI-breaking changes to upstream packages** need every affected
  consumer rebuilt against the patched headers, which only a full
  package build into the local channel gives you. aliBuild did this
  automatically.
- **Debug builds of dependencies.** conda-forge ships no debug variants.
  Configure the overlay with `-DCMAKE_BUILD_TYPE=Debug` (or your
  sanitizer flags) instead.
