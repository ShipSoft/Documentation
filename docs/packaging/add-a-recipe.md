# Add a new recipe

1. Pick a sibling recipe with a similar shape (CMake project? Header-only? Python package?) and copy its directory:

    ```bash
    cp -r recipes/random123 recipes/<new-pkg>
    ```

2. Edit `recipes/<new-pkg>/recipe.yaml`:
   - `context.version` — current upstream release.
   - `source.url` — tarball URL (GitHub `archive/refs/tags/`, GitLab
     CERN `-/archive/`, etc.). Template it on `${{ version }}` like the
     surrounding recipes so [Renovate](bump-a-version.md) can update
     it automatically.
   - `source.sha256` — `curl -fsSL <url> | sha256sum`.
   - `requirements.{build,host,run}` — declare every sibling recipe
     this package depends on by name. rattler-build resolves the
     build order from these lists, so this is the **only** place to
     express ordering.
   - Compiled packages request the sysroot in `requirements.build`
     as `if: linux` / `then: sysroot_${{ target_platform }} 2.17.*`,
     not `${{ stdlib('c') }}`. `stdlib('c')` would change the hash of
     every compiled package and rebuild the whole linux-64 channel.
   - `build.skip` — if the package isn't meant for linux-aarch64, add
     `target_platform != "linux-64"`. Otherwise the arm64 CI job
     builds it too (see [Platforms](index.md#platforms)).
   - `tests:` — at minimum a `package_contents:` block listing the
     headers / libs / CMake configs the build is expected to produce.
     For runtime-executable packages add a `script:` test that runs
     a Python import or `root -l -q` invocation.
   - `about:` — populate `homepage`, `repository`, `summary`,
     `description`, `license` (SPDX), `license_file`.
   - `extra.recipe-maintainers:` — your GitHub handle, plus at least
     one co-maintainer where possible.

3. Adjust the build script for the build system in use. Most recipes
   have a `build.sh`; noarch and simple packages (for example `ganga`,
   `genie-data`, `field-service`) put the script inline in
   `recipe.yaml` instead.

4. If the package links ROOT, add a `variants.yaml` with
   `root_cxx_standard` and the shared `root_base` pin. Copy it from a
   sibling such as `recipes/fairship/variants.yaml`; see
   [ROOT C++ standard variants](index.md#root-c-standard-variants).

5. From the repo root, build the package locally:

    ```bash
    pixi run rattler-build build --recipe recipes/<new-pkg> \
        --channel https://prefix.dev/ship-experiment/stable --channel conda-forge
    ```

6. Run the linters before you commit (see
   [Linting](#linting) below):

    ```bash
    pixi run -e lint lint
    ```

7. Open a PR. CI calls the reusable
   [`rattler-build-and-upload.yml`](ci-and-secrets.md#buildyml-build-and-upload)
   workflow, which runs
   `rattler-build build --recipe-dir recipes/ --skip-existing=all`
   and so rebuilds every recipe that isn't on the channel yet. PR
   builds don't upload; the built `.conda` files are attached to the
   run as `conda-packages-<platform>` artifacts for 7 days, so you can install
   them for testing. Uploads to `prefix.dev/ship-experiment/stable`
   happen only on `main` (push or manual dispatch).

8. After merge, decide whether the recipe is an
   [upstream candidate](https://github.com/ShipSoft/ship-conda-recipes/blob/main/README.md#packages)
   and update the table.

## Linting

`ship-conda-recipes` uses the same prek setup as the other SHiP
repositories (see [Linting & git hooks](../dev-guide/linting-and-hooks.md)).
Install the hooks once with `pixi run -e lint install-hooks`. The
hooks run:

- yamllint on every YAML file;
- shellcheck on `recipes/*/build.sh`;
- `reuse lint`, so new files need an SPDX licence header or an entry
  in `REUSE.toml`;
- the `variant-lockstep` check on `recipe.yaml` and `variants.yaml`;
- commitizen on the commit message (Conventional Commits).

## Notes

- Recipe directory name **must** equal the `package.name` in
  `recipe.yaml`. rattler-build's cross-recipe dependency resolution
  keys off the name in `requirements:`.
- Don't add the recipe to a separate manifest file: there isn't
  one. The `requirements:` block is the single source of truth for
  dependency order.
- For the recipe currently under the most churn there is a
  `build-<name>` pixi task shortcut (at the moment `build-aegir`).
  `pixi run build-all` builds the whole channel locally the same way
  CI does.
