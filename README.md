# Norway vector map for OffARO

An offline-capable vector base map of mainland Norway, built from Kartverket's **N50 Kartdata**
and published here as [PMTiles](https://github.com/protomaps/PMTiles) archives on the
[Releases](../../releases) page. [OffARO](https://github.com/OffARO/OffARO) renders it on the GPU
with its built-in N50 style and caches tiles for offline use.

## Files

Each release is one edition of the N50 data. The country is split at 63°N so that each file stays
under GitHub's 2 GiB asset limit; OffARO shows the pair as a single map.

| File | Covers | Zoom |
|---|---|---|
| `norge-sor.pmtiles` | 4.0–31.8°E, 57.7–63.0°N (south of Molde / Kristiansund) | 5–14 |
| `norge-nord.pmtiles` | 4.0–31.8°E, 63.0–71.4°N (Trondheim and north) | 5–14 |
| `SHA256SUMS.txt` | checksums of both archives | |

Tiles on the 63°N line are present in both files.

## Use in OffARO

Add a tile source with service type **Vector** and this URL (both archives, joined with `|`):

```
pmtiles://https://github.com/OffARO/norway-vector-map/releases/download/<tag>/norge-sor.pmtiles|pmtiles://https://github.com/OffARO/norway-vector-map/releases/download/<tag>/norge-nord.pmtiles
```

Replace `<tag>` with the release tag (for example `n50-2026-11`). The archives are also usable from
any MapLibre GL JS app with the `pmtiles` protocol: the vector layers are `landcover`,
`water_lines`, `water_edges`, `contours`, `spot_heights`, `transport`, `building_areas`,
`building_points`, `structure_lines`, `landcover_points`, `place_names` and `admin_lines`, with
N50's own attribute names (`objtype`, `hoyde`, `typeveg`, `vegkategori`, `vegnummer`, …).

## Detail overlays

Separate archives, each one **Overlay** tile source in OffARO (service type Vector, Overlay on),
drawn over any base map. Each has its own release tag.

| File | Release tag | Layer | Attributes | Zoom | Source |
|---|---|---|---|---|---|
| `norge-bygninger.pmtiles` | `buildings-<edition>` | `buildings` | `type` (Matrikkelen building type code), `cat` (residential · cabin · hut · outbuilding · farm · industry · commerce · public · other), `status` | 12–15 | Kartverket, *Matrikkelen - Bygningspunkt*, CC BY 4.0 |
| `norge-bygningsomriss.pmtiles` | `osm-buildings-<edition>` | `footprints` | `kind` (OSM `building=*` value), `name` | 13–16 | OpenStreetMap contributors, ODbL 1.0 |
| `norge-turruter.pmtiles` | `trails-<edition>` | `trails` | `kind` (foot · ski · bike · other), `marked`, `name`, `grade` (G · B · R · S), `imp` (1 national · 2 regional · 3 local) | 8–14 | Kartverket, *Turrutebasen*, CC BY 4.0 |

URL for a detail overlay:

```
pmtiles://https://github.com/OffARO/norway-vector-map/releases/download/<tag>/<file>
```

## Source data and build

- Data: Kartverket, *N50 Kartdata* (Geonorge, FGDB, EPSG:25833, whole country), licence
  [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Each release names the download date.
- Build: GDAL `ogr2ogr` → GeoJSONSeq per layer → `tippecanoe` (z5–14, per-layer zoom filters,
  place names in a separate pass without density dropping) → `tile-join` → `pmtiles extract` for
  the North/South split. The script lives in OffARO's source tree as `scripts/vector-map/build-n50.sh`.
- Buildings: Kartverket, *Matrikkelen - Bygningspunkt* (Geonorge, FGDB, CC BY 4.0) → `ogr2ogr`
  (type, status, category) → `tippecanoe` z12–15; script `scripts/vector-map/build-buildings.sh`.
- Building footprints: OpenStreetMap via Geofabrik `norway-latest.osm.pbf` → `osmium tags-filter`
  `wr/building` → `osmium export` (polygons, tags `building`, `name`) → `tippecanoe` z13–16; script
  `scripts/vector-map/build-osm-buildings.sh`.
- Trails: Kartverket, *Turrutebasen* (Geonorge, FGDB, CC BY 4.0) → `ogr2ogr` into an indexed
  GeoPackage (the route facts live in separate info tables) → one `trails` layer → `tippecanoe`
  z8–14; script `scripts/vector-map/build-trails.sh`.

## Licence and attribution

The archives are derived from Kartverket data (N50 Kartdata, Matrikkelen, Turrutebasen) and are published under the same licence,
**Creative Commons Attribution 4.0** (CC BY 4.0). Attribute the data as **© Kartverket**.
The archive metadata carries the same attribution and OffARO shows it on the map.

The OSM footprints archive (`norge-bygningsomriss.pmtiles`, release tags `osm-buildings-*`) is a
separate work under the [Open Database License 1.0](https://opendatacommons.org/licenses/odbl/1-0/)
(ODbL), **© OpenStreetMap contributors**; it is never merged with the CC BY archives.
