# Troubleshooting

## `lock-file not up-to-date` in an unrelated environment

```
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
released `shipgeometryservice` requires `shipdatamodel >=0.1.0,<0.2`).
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

## Where things live

| Artifact | Location |
| --- | --- |
| Source-dependency build dirs | `<consumer>/.pixi/bld/<package>/` |
| Local channel | `../ship-local-channel/{linux-64,noarch}/` |
| Dev environments | `<consumer>/.pixi/envs/dev/` |

`pixi clean` in the consumer removes `.pixi` (environments and build
dirs) if you want a fully fresh start.
