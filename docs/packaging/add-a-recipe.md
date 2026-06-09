# Add a new recipe

1. Pick a sibling recipe with a similar shape (CMake project? Header-only? Python package?) and copy its directory:

    ```bash
    cp -r recipes/random123 recipes/<new-pkg>
    ```

2. Edit `recipes/<new-pkg>/recipe.yaml`:
   - `context.version` — current upstream release.
   - `source.url` — tarball URL (GitHub `archive/refs/tags/`, GitLab
     CERN `archive`, etc.). Use the same template variables as the
     surrounding recipes so [Renovate](bump-a-version.md) can update
     it automatically.
   - `source.sha256` — `curl -fsSL <url> | sha256sum`.
   - `requirements.{build,host,run}` — declare every sibling recipe
     this package depends on by name. rattler-build resolves the
     build order from these lists, so this is the **only** place to
     express ordering.
   - `tests:` — at minimum a `package_contents:` block listing the
     headers / libs / CMake configs the build is expected to produce.
     For runtime-executable packages add a `script:` test that runs
     a Python import or `root -l -q` invocation.
   - `about:` — populate `homepage`, `repository`, `summary`,
     `description`, `license` (SPDX), `license_file`.
   - `extra.recipe-maintainers:` — your GitHub handle, plus at least
     one co-maintainer where possible.

3. Adjust `recipes/<new-pkg>/build.sh` for the build system in use.

4. From the repo root, dry-run the build locally:

    ```bash
    pixi run rattler-build build --recipe recipes/<new-pkg> \
        --channel https://prefix.dev/ship --channel conda-forge
    ```

5. Open a PR. CI runs `pixi run build-all`
   (`rattler-build build --recipe-dir recipes/ --skip-existing=all`),
   which rebuilds every recipe whose dependencies changed. Uploads to
   `prefix.dev/ship` are gated on push to `main`.

6. After merge, decide whether the recipe is an
   [upstream candidate](https://github.com/ShipSoft/ship-conda-recipes/blob/main/README.md#packages)
   and update the table.

## Notes

- Recipe directory name **must** equal the `package.name` in
  `recipe.yaml`. rattler-build's cross-recipe dependency resolution
  keys off the name in `requirements:`.
- Don't add the recipe to a separate manifest file — there isn't
  one. The `requirements:` block is the single source of truth for
  dependency order.
- For a single-recipe local iteration, run the inline form:

    ```bash
    pixi run rattler-build build --recipe recipes/<new-pkg> \
        --channel https://prefix.dev/ship --channel conda-forge
    ```

  or, for the recipe currently most under churn, the `build-<name>`
  pixi task shortcut.
