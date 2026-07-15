# Pennsylvania Substation Flood Exposure

A geospatial analysis of flood exposure for roughly 2,333 electrical substations across Pennsylvania. Each substation is linked to the stream network, assigned a scaled peak discharge, and flagged for whether it sits inside a FEMA floodplain and/or below the Base Flood Elevation (BFE).

## Headline results

- **196 of 2,333 substations (8.4%)** fall inside a FEMA Special Flood Hazard Area.
- **295 of 2,333 (12.6%)** sit below the Base Flood Elevation.
- **18** are both inside the floodplain and below BFE.

Exposure concentrates in the Berks / Northampton / Montgomery corridor (floodplain) and around Erie, Philadelphia, and Delaware counties (below BFE).

## Data sources

| Layer | Source |
|---|---|
| Substation locations | OpenStreetMap (via `osmnx`) |
| Stream gages and annual peak flows | USGS NWIS (peak-flow and site web services) |
| Stream network, catchments, drainage area | NHDPlus (EPA 2022 snapshot, Pennsylvania) |
| Floodplains and Base Flood Elevation | FEMA (Special Flood Hazard Areas / Flood Hazard Areas) |
| Ground elevation | Digital Elevation Model (`watershed_dem/dem.tif`) |

## Method in brief

1. Pull USGS gages and annual peak flows; compute exceedance probabilities with the Weibull plotting position, `rank / (n + 1)`.
2. Attach each substation to its nearest NHD flowline and hydro-rank candidates by drainage area.
3. Scale a gage's discharge to the substation location by the drainage-area-ratio method: `Q_sub = Q_gage * (A_sub / A_gage) ^ 0.8`.
4. Sample the DEM, compare ground elevation against BFE, and overlay FEMA Special Flood Hazard Areas.
5. Classify each substation into a four-class exposure scheme (floodplain x below-BFE) and build presentation maps and county summaries.

## Repository layout

```
notebooks/         analysis, in run order (01 -> 04)
arcgis_project/     ArcGIS Pro project (.aprx) and toolbox (.atbx)
results/
  tables/          county and operator exposure summaries (CSV, XLSX)
  figures/         analytical plots (distributions, match diagnostics)
  maps/            exported result maps (statewide and city detail)
```

## Notebook workflow

| Notebook | Purpose |
|---|---|
| `01_usgs_gages_flood_frequency.ipynb` | Pull USGS gages and peak flows; flood-frequency / exceedance calculations; match substations to gages. |
| `02_flowline_linking_discharge_bfe.ipynb` | Link substations to NHD flowlines; drainage-area discharge scaling; DEM and BFE integration; build the composite feature class. |
| `03_fema_bfe_crossref_summaries.ipynb` | FEMA floodplain cross-reference, non-SFHA distance stats, summary tables and Excel export. |
| `04_presentation_maps.ipynb` | Four-class exposure classification, presentation feature classes, map layers, and county summaries. |

## Running locally

The notebooks reference two local folders through environment variables (with editable fallbacks in the first cell of each notebook):

- `PA_FLOOD_PROJECT` — folder holding `PA Flood Risk GIS.gdb` and the result outputs.
- `PA_FLOOD_DATA` — folder holding the NHDPlus geodatabase, the DEM, and the OSM data.

Copy `.env.example` to `.env` and set these (or edit the fallback strings directly). See `.env.example` for detail.

## Data availability

The raw geospatial data (the ~2 GB File Geodatabase, NHDPlus, DEM) is **not** committed to this repository; it is stored separately. The notebooks and result artifacts here document the full workflow and outputs. The included `.aprx` embeds absolute paths to that data and is provided as a record rather than a runnable project.

## Portability

This analysis was built in ArcGIS Pro using `arcpy`. For a plan to migrate it to open-source QGIS / Python (GeoPandas, rasterio), see [`MIGRATION_QGIS.md`](MIGRATION_QGIS.md).

## Tools

Python (`arcpy`, `geopandas`, `pandas`, `numpy`, `requests`, `osmnx`, `matplotlib`), ArcGIS Pro, USGS NWIS, NHDPlus, FEMA NFHL.
