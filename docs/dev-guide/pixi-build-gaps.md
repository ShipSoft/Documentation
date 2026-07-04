# pixi-build gap log

`pixi-build` is a preview feature; this page tracks the gaps and bugs
we hit while building the SHiP dev workflow on it, the workarounds in
our manifests (marked `GAP:<n>` in comments where applicable), and the
upstream state. Re-verify entries when pixi or the build backends
release — last full check: pixi 0.68.1 / 0.72.0, pixi-build-cmake 0.x,
pixi-build-rattler-build 0.3.13 & 0.4.3, 2026-07-04.

Upstream trackers:
[prefix-dev/pixi](https://github.com/prefix-dev/pixi/issues) ·
[prefix-dev/pixi-build-backends](https://github.com/prefix-dev/pixi-build-backends/issues)

| # | Area | Symptom | Workaround | Status |
| --- | --- | --- | --- | --- |
| 1 | rattler-build backend | Workspace channels are not forwarded to the recipe's build/host dependency resolution — any recipe with real requirements fails with `No candidates were found for gxx_linux-64` (only the local `file://…/output/` channel is passed). Reproduced with backend 0.3.13 (API v4, pixi 0.68.1) and 0.4.3 (API v5, pixi 0.72.0), both via `pixi build` and via a consumer path dependency. `[package.build] channels` only selects where the *backend* comes from and does not help. | field-service uses a local channel (`build-local-channel` task) instead of a source dependency. | **To file upstream** (minimal repro: any `pixi-build-rattler-build` package whose recipe has `build: [cmake]`). |
| 2 | pixi core | `--locked` validates **all** environments, so a committed dev environment with source paths breaks `pixi run --locked` for anyone without the sibling checkouts (CI!), even when only the default environment is used. | Dev blocks are committed commented-out; developers uncomment locally and never commit the result. | To raise upstream: per-environment lock validation, or an "optional environment" concept. |
| 3 | pixi core | No local/untracked manifest overlay (à la `Cargo.toml` + `.cargo/config` or npmrc): any opt-in to a dev environment means editing the committed `pixi.toml`. | Commented block + `git checkout pixi.toml pixi.lock` etiquette. | To raise upstream. |
| 4 | cmake backend | No way to author `run_exports` for the built package, so source-built packages don't pin their consumers the way released packages do. Harmless for direct source consumers (they pin by path), but transitive binary consumers lose the pin. | Version-lockstep rule; ABI caution documented. | Known limitation. |
| 5 | backends | Multi-output recipes are only possible via the rattler-build backend (blocked by gap 1). The cmake backend can't express the field-service core/g4/tools split at all. | Local channel for field-service. | Blocked on gap 1. |
| 6 | backends | Central variant config (e.g. our `root_cxx_standard` matrix) can't be consumed from `ship-conda-recipes`; values are hardcoded in the dev twins (`cxx_standard` context / `variants.yaml`). | Comments cross-reference the canonical recipe. | Known limitation. |
| 7 | pixi core | Workspace dependencies are not inherited by the `[package]` tables, so host dependency lists are duplicated between `[dependencies]` and `[package.host-dependencies]`. | Keep both lists adjacent in the manifest. | [Documented limitation](https://pixi.prefix.dev/latest/build/workspace/). |
| 8 | ecosystem | Local rattler-build runs lack conda-forge's global pinning, so free solves can pick incoherent host sets (observed: spdlog 1.10 + fmt 12 + root_base 6.36 for the phlex dev build). | `recipes-dev/phlex/variants.yaml` pins ABI-relevant packages to aegir's lock. | By design (feedstock CI concern); note kept for awareness. |

## What works well (for the record)

- Source dependencies with the **cmake backend**: `data-model`,
  `geometry`, `geometry_service` build correctly, package contents
  match the released packages, `run_exports` of host deps (ROOT) are
  picked up automatically.
- **Feature merge**: a `dev` feature's source spec coexists with the
  default feature's binary spec for the same package; the source spec
  wins in the merged environment, other environments are untouched
  (verified in the lock: `conda_source: shipdatamodel … @ ../data-model`
  in `dev`, released `.conda` elsewhere).
- **Relative paths** in specs and lock entries — portable across
  machines that follow the sibling-checkout layout.
- **Rebuild loop**: edit → `pixi run -e dev build` ≈ 20–30 s for
  data-model → aegir (change detection + package rebuild + reinstall +
  incremental consumer build); no-op runs < 1 s.
- **Relative local channels**: `channels = [{ channel =
  "../ship-local-channel", priority = 10 }]` in a feature works
  (the lock then records absolute `file://` URLs — one more reason
  those blocks stay uncommitted).
