# Migrating this project from ArcGIS / arcpy to QGIS / open source

This project was built in ArcGIS Pro and depends on `arcpy`, which only runs with a licensed ArcGIS install. This document is a plan for reproducing the workflow with open-source tools (QGIS and the Python geospatial stack) so the analysis survives losing ArcGIS access.

The short version: the tabular and hydrology work ports with little or no change, the spatial operations need a rewrite into GeoPandas / rasterio, and the ArcGIS project file (maps and symbology) does not transfer and must be rebuilt.

## The one time-sensitive action

**While you still have ArcGIS, export the feature classes you care about from the File Geodatabase to GeoPackage (`.gpkg`).** GeoPackage is QGIS's native format and is read and written cleanly by GeoPandas.

You do not strictly have to, because GDAL's `OpenFileGDB` driver lets GeoPandas read a File Geodatabase directly:

```python
import geopandas as gpd
gdf = gpd.read_file("PA Flood Risk GIS.gdb", layer="PA_substations_composite_final")
```

Reading `.gdb` is reliable. Writing back to `.gdb` is not the goal; write to `.gpkg` instead. Exporting to `.gpkg` now is still worth doing as a clean, self-contained snapshot of the data.

## What transfers as-is

These parts are already pure `pandas` / `numpy` / `requests` / `geopandas` and need no changes:

- The USGS NWIS pull (peak flows and site metadata over HTTP).
- Flood-frequency / exceedance calculations (`rank / (n + 1)`).
- Drainage-area discharge scaling (`Q_sub = Q_gage * (A_sub / A_gage) ^ 0.8`).
- Any `pandas` groupby, summary, and Excel/CSV export logic.

## What must be rewritten (arcpy to open source)

Every `arcpy` call in the notebooks has a direct open-source equivalent:

| arcpy | Open-source replacement |
|---|---|
| `arcpy.analysis.SpatialJoin` | `geopandas.sjoin` |
| `arcpy.analysis.Near`, `GenerateNearTable` | `geopandas.sjoin_nearest` (GeoPandas >= 0.10), or shapely `.distance` |
| `arcpy.analysis.Select` (SQL `where_clause`) | boolean mask / `gdf.query(...)` |
| `arcpy.management.Project` | `gdf.to_crs(epsg=...)` |
| `arcpy.analysis.Clip` | `geopandas.clip` |
| `arcpy.analysis.Statistics` | `gdf.groupby(...).agg(...)` |
| `arcpy.management.AddField` / `CalculateField` / `UpdateCursor` | pandas column assignment |
| `arcpy.da.SearchCursor` (row iteration) | iterate the GeoDataFrame, or vectorize |
| `arcpy.sa.ExtractMultiValuesToPoints` (DEM sampling) | `rasterio` `.sample()` or the `rasterstats` `point_query` |
| `arcpy.management.Merge` | `pandas.concat` / `geopandas` concat |
| `arcpy.management.CopyFeatures` | `gdf.to_file(..., driver="GPKG")` |

Notes:

- The CRS in this project is `NAD_1983_2011_StatePlane_Pennsylvania_South_FIPS_3702_Ft_US`. Confirm its EPSG code (2272 appears in layer names, which is PA South, US survey feet) and set it explicitly with `to_crs`. Distances in that CRS are in **feet**, which the `Near` results assume; keep units consistent after the rewrite.
- `sjoin_nearest` returns the nearest feature and, with `distance_col=`, the distance. That replaces both `Near` and `GenerateNearTable`.
- DEM sampling: `rasterstats.point_query(points_gdf, "dem.tif")` returns elevation per point, replacing `ExtractMultiValuesToPoints`.

## What does not transfer

- The **`.aprx` project** (maps, layouts, symbology, layer definitions) cannot be opened in QGIS. You would rebuild the maps as a QGIS project (`.qgz`) and recreate symbology by hand. The four-class and floodplain/BFE styling is simple categorical symbology, so this is straightforward but manual.
- The presentation-map notebook (`04_...`) is almost entirely `arcpy.mp` map-automation code. In QGIS the equivalent is PyQGIS (`qgis.core`), but for a one-time rebuild it is usually faster to style the layers in the QGIS GUI than to script it.

## Suggested migration order

1. Export the key feature classes from the `.gdb` to `.gpkg` (or plan to read the `.gdb` directly).
2. Set up an environment: `pip install geopandas rasterio rasterstats pyogrio osmnx matplotlib`.
3. Port notebook `02` first — it is the analytical core (flowline linking, discharge scaling, DEM/BFE). Everything downstream depends on its composite output.
4. Port `03` (FEMA cross-reference and summaries), which is mostly joins and groupbys.
5. Recreate the maps from `04` in the QGIS GUI rather than porting the automation code.
6. Validate: compare a few substations' `q_sub_scaled`, `bfe_flag`, and floodplain flags against the ArcGIS results to confirm the rewrite matches.

## Things to test carefully

- CRS handling and distance units (feet vs meters) after `to_crs`.
- Nearest-neighbor ties: `sjoin_nearest` and `Near` may break ties differently; spot-check substations that sit between two flowlines.
- Reading the `.gdb` with `pyogrio` vs `fiona` back-ends can differ in field names and null handling; verify key fields (`totdasqkm`, `gnis_name`, `ftype`) come through intact.
