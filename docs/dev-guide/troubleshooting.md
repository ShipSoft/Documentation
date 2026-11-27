# Troubleshooting

## `lock-file not up-to-date` in an unrelated environment

```text
× lock-file not up-to-date with the workspace
```

after uncommenting a dev block usually means pixi can't find a sibling
checkout (`the source path '…' could not be found`): pixi validates
*every* environment against the lock, even ones you never use. Either
create the missing checkout, trim the source-dependency list, or — to
run a default-environment task untouched — use `--frozen` instead of
`--locked`.

## Solver conflicts when mixing source and binary packages

Binary SHiP packages pin each other through `run_exports` (e.g. the
released `shipgeometryservice` requires `shipgeometry >=0.2.1,<0.3`,
and the released aegir 0.4.0 requires `shipdatamodel >=0.5.0,<0.6`).
If a source checkout's `[package].version` has drifted from the last
release, those pins become unsatisfiable and the solve fails (or
silently keeps the binary package). Fix the version in the checkout's
`[package]` section — and if you're the one releasing, remember the
[lockstep rule](multi-package.md#rules).

## Changes in the dependency don't show up

- Confirm the environment: `pixi list -e dev <package>` should show
  the path (or `file://…` channel) instead of `https://prefix.dev/…`.
- Source-dependency rebuilds trigger on the next `pixi run -e dev …`;
  if you suspect staleness, `pixi update <package>` (or as a big
  hammer `rm -rf .pixi && pixi install -e dev`).
- Local-channel packages (field-service, phlex) are *not* rebuilt
  automatically — re-run `build-local-channel`/`build-phlex-pr` after
  each change, then `pixi update <packages>` in the consumer.

## Runtime crashes after mixing source and binary packages

The binary packages were compiled against *released* headers. If your
source checkout changes ABI (class layout, virtual tables, inline
functions used across package boundaries), binary consumers of that
package will misbehave at runtime. Build the affected consumers from
source too, or via the local channel.

## `Failed to resolve dependencies` while building a source dependency

If the failing spec is a build tool (`gxx_linux-64`, `cmake`, …) and
the listed channels show only a `file://…/output/` entry, you've hit
the rattler-build-backend channel bug — see the
[gap log](pixi-build-gaps.md). Affected packages must go through the
local channel for now.

## `excluded because due to strict channel priority`

A dev solve that fails like this, naming a package you didn't build
locally on purpose, usually means an old build of it is still in
`../ship-local-channel`. pixi uses strict channel priority, so any
version of a package in the local channel hides all versions of it in
the remote channels. Find the builds of the package the error names,
delete them and re-index. For example, for a stale field-service 0.2.0:

```console
$ ls ../ship-local-channel/*/field-service-*
$ rm ../ship-local-channel/linux-64/field-service-0.2.0-*.conda
$ pixi exec -s rattler-index -- rattler-index fs ../ship-local-channel
```

Use the package name and version from the error, and the platform
directory (`linux-64`, `noarch`, …) that `ls` shows them in.

If you don't need the local channel at all, leave the `channels` line
of the dev block commented out.

The same message appears when a source dependency doesn't satisfy a
version range declared elsewhere in the manifest, for example a
`data-model` checkout at 0.7.0 in an environment that also has
`shipdatamodel >=0.6,<0.7`. pixi then names only the channel builds,
not the source build that actually failed (gap 10 in the [gap
log](pixi-build-gaps.md)). Check the `[package].version` of the
checkout against the specs the `dev` environment includes.

## Where things live

| Artifact | Location |
| --- | --- |
| Source-dependency build dirs | `<consumer>/.pixi/bld/<package>/` |
| Local channel | `../ship-local-channel/{linux-64,noarch}/` |
| Dev environments | `<consumer>/.pixi/envs/dev/` |
| Overlay build dir | `<src>-overlay-build/` next to the source checkout |
| Overlay records | `<consumer>/.pixi/envs/<env>/.ship-overlay/<package>/` |

`pixi clean` in the consumer removes `.pixi` (environments and build
dirs) if you want a fully fresh start.
