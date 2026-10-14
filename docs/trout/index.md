# Trout

Trout is the reconstruction package for the SHiP experiment. It runs on the
output of the [Shannon](../shannon/index.md) digitisation, or on real detector
data. Track finding and fitting in the spectrometer use
[ACTS](https://github.com/acts-project/acts).

Repository: [ShipSoft/Trout](https://github.com/ShipSoft/Trout)

API reference: [Doxygen](https://shipsoft.github.io/Trout/)

## How it works

Trout is a set of [Phlex](https://github.com/Framework-R-D/phlex) plugins
configured with Jsonnet workflows (stage `reconstruction`). The shared
configuration lives in `workflows/lib/reconstruction.libsonnet`:

| Plugin | Role |
|--------|------|
| `read_digitised_hits` | Reads Shannon's digitised hit RNTuples |
| `acts_geometry_provider` | Builds the ACTS tracking geometry from a GeoModel `.db` file ([Geometry](../geometry/index.md), [Geometry Service](../geometry_service/index.md)) |
| `acts_field_provider` | Provides the spectrometer field map to ACTS through the [Field Service](../field_service/index.md) |
| `spectrometer_tracking` | Seeds, finds and fits tracks in the straw-tube spectrometer |
| `upstream_tagger_reco`, `surround_tagger_reco`, `calorimeter_reco`, `timing_detector_reco` | Per-detector reconstruction (only with `full_reco`) |
| `rec_output_module` | Writes each reconstructed collection to its own RNTuple, with optional validation histograms |

Two workflows are provided:

- `workflows/run_reconstruction.jsonnet` runs the full reconstruction.
- `workflows/tracking.jsonnet` runs spectrometer tracking only (`full_reco: false`).

Both read `../shannon/digitised_hits_time.root` and write
`track_fit_results.root`. The geometry database defaults to
`../geometry/my_ship_geometry.db`; override `input_file`, `db_file` and the
field map for your setup.

## Building

```bash
git clone https://github.com/ShipSoft/Trout.git
cd Trout
pixi run build
pixi run test
pixi run smoke     # generate inputs in-tree and run workflows/smoke.jsonnet
```

Trout depends on data-model, Geometry, geometry_service, field_service,
ACTS and ship-random, all resolved from conda-forge and
[prefix.dev/ship](https://prefix.dev/channels/ship). It is packaged as `trout`
on that channel.
