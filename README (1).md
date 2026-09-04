# Wolfach Flash Flood Susceptibility Mapping

DEM-based flash flood susceptibility analysis for the Wolfach sub-watershed, Ortenaukreis, Baden-Württemberg — Germany's highest flood-risk district. The workflow combines QGIS (SAGA hydrology tools) and ArcGIS Pro to derive a watershed-scale, 1–5 classified flood susceptibility map, validated against LUBW historical flood zones.

## Datasets

| Dataset | Source | Notes |
|---|---|---|
| DGM1 (Digital Terrain Model, 1 m) | 72 tiles, LGL Baden-Württemberg | Merged into a single seamless DEM |
| Wolfach river network | Baden-Württemberg river system dataset | Used for stream reference and pour point placement |
| Settlements in the basin | Geofabrik — `baden-wuerttemberg-260719.osm.pbf` | Opened and filtered in QGIS, then transferred to ArcGIS |

## Tools

- **QGIS** (SAGA & GDAL providers) — DEM preprocessing and initial hydrology (Steps 1–8)
- **ArcGIS Pro** (Spatial Analyst) — Watershed delineation, risk factor derivation, and overlay analysis (Steps 9–14)

## Workflow

### Part 1 — QGIS (Steps 1–8)

**1. Organize raw tiles**
All 72 `.tif` DGM1 tiles placed in a single folder (e.g. `C:\GIS\Wolfach\DGM1_raw`), kept free of unrelated files.

**2. Visual inspection**
All tiles loaded via *Layer → Add Layer → Add Raster Layer* and checked on the canvas to confirm they align edge-to-edge with no gaps.

**3. Merge tiles**
*Raster → Miscellaneous → Merge*, all 72 tiles as input, output CRS set to **ETRS89 / UTM32N (EPSG:25832)**, exported as `wolfach_merged_dgm1.tif`.

> **Memory-safe alternative:** merging 70+ large tiles at once can exceed available RAM (`numpy._core._exceptions._ArrayMemoryError`). The fix: build a **Virtual Raster** first (*Raster → Miscellaneous → Build Virtual Raster*, near-instant, no memory cost), then convert it to a real GeoTIFF with *Raster → Conversion → Translate*, adding creation options `COMPRESS=LZW`, `TILED=YES` (and `BIGTIFF=YES` if output exceeds 4 GB). This processes the raster block-by-block instead of loading it entirely into memory.

**4. Clip to study area (optional)**
A rough polygon drawn by hand around the Wolfach valley, used with *Raster → Extraction → Clip Raster by Mask Layer* to reduce file size before hydrology processing.

**5. Fill Sinks**
*SAGA → Fill Sinks (Wang & Liu)* run on the merged/clipped DEM to remove spurious depressions that would otherwise corrupt flow direction.

**6. Flow Direction & Accumulation**
*SAGA → Catchment Area (Flow Accumulation)*, method **Deterministic 8 (D8)**, run on the filled DEM — computes both flow direction and flow accumulation in one step.

**7. Stream network extraction**
Flow accumulation thresholded via Raster Calculator (e.g. `"flow_acc@1" > 1000`, threshold tuned by inspection) to produce a binary stream network raster.

**8. Watershed basins (initial)**
*SAGA → Watershed Basins* run on the flow direction raster to get a first approximation of the basin boundary.

### Part 2 — ArcGIS Pro (Steps 9–14)

**9. Precise watershed delineation**
- A **pour point** placed manually at the Wolfach's lowest/outlet point, directly on a high-accumulation stream pixel
- Converted to raster: *Conversion Tools → To Raster → Point to Raster*
- *Spatial Analyst Tools → Hydrology → Watershed* run using the flow direction raster and the pour point raster → produces the hydrologically correct basin boundary

**10. Watershed to polygon & final clip**
- *Conversion Tools → From Raster → Raster to Polygon* (simplify disabled for an accurate boundary); multiple output polygons dissolved down to the main basin polygon
- Original DEM clipped to this polygon via *Data Management Tools → Raster → Clip*, with **"Use Input Features for Clipping Geometry"** enabled for a pixel-precise (non-rectangular) result → final analysis DEM

**11. Risk factor derivation** (all recomputed on the clipped DEM for speed and accuracy)
- **Slope** — *Spatial Analyst Tools → Surface → Slope* (degrees)
- **Flow Accumulation** — recalculated (Fill → Flow Direction → Flow Accumulation) on the clipped DEM
- **Distance to Stream** — stream network clipped to the watershed, converted to polyline, then *Spatial Analyst Tools → Distance → Euclidean Distance*

**12. Reclassification (1–5 risk scale)**
Each factor reclassified independently (*Spatial Analyst Tools → Reclass → Reclassify*, Natural Breaks or manual, 5 classes):
- **Slope:** low slope → high risk (5), steep slope → low risk (1) — flatter terrain accumulates water
- **Flow Accumulation:** higher accumulation → higher risk (5)
- **Distance to Stream:** closer to stream → higher risk (5)

**13. Weighted Overlay**
*Spatial Analyst Tools → Overlay → Weighted Overlay*, combining the three reclassified layers:

| Factor | Weight |
|---|---|
| Distance to Stream | 40% |
| Flow Accumulation | 35% |
| Slope | 25% |

Output: final **Flash Flood Susceptibility Map**, classified 1 (low risk) to 5 (high risk).

**14. Validation**
Final susceptibility map cross-checked against the actual Wolfach river course and, where available, LUBW's historical flood zone layer to confirm high-risk areas align with documented flooding.

## Summary

| Stage | Platform | Key Output |
|---|---|---|
| Tile merge & DEM prep | QGIS | Seamless DEM (EPSG:25832) |
| Sink fill, flow direction/accumulation | QGIS (SAGA) | Hydrologically corrected DEM, flow rasters |
| Precise watershed delineation | ArcGIS Pro | Watershed polygon, clipped DEM |
| Risk factor analysis | ArcGIS Pro | Slope, Flow Accumulation, Distance to Stream |
| Weighted Overlay | ArcGIS Pro | Flash Flood Susceptibility Map (1–5) |
| Validation | ArcGIS Pro | Cross-check vs. LUBW flood zones |
