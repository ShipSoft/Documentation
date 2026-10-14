# Configuration

FairShip builds its geometry from `python/geometry_config.py`, which
reads a set of YAML files from `$FAIRSHIP/geometry/` and takes a few
options from the command line of `run_simScript.py` and
`run_fixedTarget.py`.

## Geometry YAML files

| File | Content |
| --- | --- |
| `target_config.yaml` | Target layout (plates, materials, lengths). Default for both macros. |
| `target_config_2026.yaml` | 2026 BDF target design (tungsten disks in a steel core). Select it explicitly. |
| `veto_config_helium.yaml`, `veto_config_vacuums.yaml` | Decay vessel and veto layout. Chosen from the decay volume medium. |
| `strawtubes_config.yaml` | Straw tube tracker (SST) layout. |
| `MTC_config.yaml`, `SiliconTarget_config.yaml` | SND detectors (MTC and silicon target). |

To try a different layout, copy the file, edit it and pass the copy on
the command line instead of changing the default in place.

## Command-line options

`run_simScript.py`:

- `--target-yaml <file>` selects the target configuration, for example
  `--target-yaml $FAIRSHIP/geometry/target_config_2026.yaml`.
- `--helium` (default) or `--vacuums` sets the decay volume medium and
  with it the veto YAML file.
- `--shieldName` picks the muon shield from the database (`TRY_2026` by
  default, or `TRY_2025`).
- `--SND` turns on the SND; `--SND_design` chooses the design
  (`1`: emulsion target, `2`: MTC and silicon target, or `all`).
- `--field_map files/<name>.root` selects the spectrometer field map.
- `--strawDesign` sets the tracker station frame material (`10`: steel, the default; `4`: aluminium).
- `-g <geofile>` reuses the geometry of an earlier run (on `main`, after
  26.09). The geometry options above, apart from `--field_map`, cannot be
  combined with it (see [Running](running.md#reusing-a-geometry)).

`run_fixedTarget.py` has the equivalent `--TARGET_YAML`, `--shieldName`
and `--DecayVolumeMedium` options, plus options for the beam position
and spread (`--x-offset`, `--y-offset`, `--z-offset`, `--beam-smear`,
`--beam-paint`).

Use `--help` on either macro for the full list and the defaults.
