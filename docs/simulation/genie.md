# GENIE neutrino generation

[aegir-genie](https://github.com/ShipSoft/aegir-genie) generates neutrino
interactions with [GENIE](https://www.genie-mc.org/) inside an aegir job. It
is a phlex source plugin, `genie_source`, that publishes `MCParticle` vectors
like aegir's other generators.

FairShip runs GENIE as a chain of command-line steps (`gmkspl` → `gevgen` →
`gntpc` → vertex re-placement). The plugin drives a `genie::GMCJDriver`
directly instead. It convolves the SHiP neutrino flux with the cross-section
splines and the GeoModel geometry, so interaction vertices follow the same
material that Geant4 tracks through, and it keeps count of protons on target.

Repository: [ShipSoft/aegir-genie](https://github.com/ShipSoft/aegir-genie)

API reference: [Doxygen](https://shipsoft.github.io/aegir-genie/)

## Why a separate repository

GENIE is GPL-licensed, so anything that links it becomes a combined GPL work.
aegir is LGPL-3.0-or-later and must not depend on GENIE, which is why
everything that links GENIE lives in aegir-genie. The SHiP-authored sources
there are LGPL-3.0-or-later as well; only the plugin binary, linked against
GENIE, is distributed under GPL-3.0-or-later. aegir never links this code.
Phlex finds the plugin at run time through `PHLEX_PLUGIN_PATH`.

aegir itself contains `genie_reader_source`, which reads GENIE rootracker
files with plain ROOT I/O. That path needs no GENIE libraries and serves as
the validation reference for the embedded source.

## Installing

aegir-genie and the production cross-section splines are packaged on the
[prefix.dev/ship](https://prefix.dev/channels/ship) channel:

```bash
pixi workspace channel add https://prefix.dev/ship
pixi add aegir aegir-genie genie-splines-ship
```

`genie-splines-ship` installs the splines as
`$CONDA_PREFIX/share/genie-splines-ship/gxspl-ship.xml`. The example
workflows are installed under `$CONDA_PREFIX/share/aegir-genie/workflows/`.

To build from source:

```bash
git clone https://github.com/ShipSoft/aegir-genie.git
cd aegir-genie
pixi run build
pixi run test
```

## Running

Two example workflows are included:

- `workflows/genie_st.jsonnet`: a short single-threaded chain with a
  schema-v1 SHiP flux file.
- `workflows/genie_ship_full.jsonnet`: the full chain on the production
  geometry with the neutrino group's GSimple flux, as used for validation.

Both need a flux file, the spline XML and a GeoModel geometry database. The
geometry, Geant4 and output plugins in these workflows come from aegir, so
run full chains from an environment that has aegir and its Geant4 data, with
aegir-genie's plugin on `PHLEX_PLUGIN_PATH` and `GENIE` pointing at
`share/genie`. The
[aegir-genie README](https://github.com/ShipSoft/aegir-genie#running) covers
the configuration keys, both flux formats and the `gevgen_ship` validation
tool.
