# sea-cucumber

sea_cucumber is the SHiP event display. It reads a GeoModel geometry
database (`.db`), an RNTuple file in the SHiP [data model](../data-model/index.md)
and an optional TOML view config. It links no analysis or framework code, so
it works on output from any stage.

There are two viewers sharing one core:

- a web frontend (three.js) that runs in the browser, the main interface;
- a ROOT REve display, kept as a second option.

Repository: [ShipSoft/sea-cucumber](https://github.com/ShipSoft/sea-cucumber)

Full reference: [manual](https://github.com/ShipSoft/sea-cucumber/blob/main/docs/manual.md)

The project is in alpha: both viewers work for day-to-day event inspection,
but interfaces may still change.

## Building

```bash
git clone https://github.com/ShipSoft/sea-cucumber.git
cd sea-cucumber
pixi run build
pixi run test
```

## Web frontend

Produce the display files, then serve them:

```bash
pixi run web-data --geometry ship_geometry.db --data output.root --view views/default.toml
pixi run web        # open http://localhost:8080
```

`web-data` writes `geometry.json`, one `event_<i>.json` per event and
`manifest.json` into `web/data/`. Useful flags are `--out <dir>`,
`--events all|<i>`, `--depth <n>` and `--max-shapes <n>`.

## ROOT REve display

```bash
pixi run run --geometry ship_geometry.db --data output.root --view views/default.toml --event 0
```

A bare `--geometry` filename is looked up in the current directory, then in
`$SHIPGEOMETRY_ROOT/share/geometry/`, as in aegir. Add `--inspect <depth>` to
print where subsystems sit in z without loading any data.

Reading a large `.db` takes seconds on every launch. Convert it once with
`pixi run geo-cache --geometry ship_geometry.db --view views/default.toml`,
then pass `--geo-cache geocache` to the display.

## Data read

The display reads the `events` RNTuple fields `mc_particles`, `sim_hits`,
`sim_particles`, `rec_particles` and `sim_result`. Any of them may be
missing. Dense subsystems such as straws and tiles should be filtered with
`include`/`exclude` regexes in the view config to keep both viewers
responsive.
