# Methodology

## 1. Project Overview

This project develops a **DEM-based flash flood susceptibility map** for the Wolfach
sub-catchment in Ortenaukreis, Baden-Württemberg — the district with the highest
flash flood risk in Germany. The analysis follows a raster hydrology workflow in
Esri ArcGIS Pro (Spatial Analyst), combining terrain and hydrological factors into
a single weighted suitability surface.

## 2. Workflow Overview

The processing chain follows a standard DEM-based hydrology pipeline:

```
DEM (DGM1) → Fill → Flow Direction → Flow Accumulation → Watershed Delineation
   → Slope → Distance to Stream → Reclassify → Weighted Overlay
```

## 3. Step-by-Step Process

### 3.1 DEM Preparation
- Source: DGM1 (1 m resolution digital terrain model), LGL-BW
- Tiles mosaicked and clipped to the Wolfach sub-catchment boundary

### 3.2 Hydrological Conditioning
- **Fill** — removes sinks from the DEM to ensure continuous flow paths
- **Flow Direction** — computes the direction of steepest descent per cell (D8 method)
- **Flow Accumulation** — computes the number of upstream cells draining into each cell
- **Watershed Delineation** — derives the Wolfach sub-catchment boundary from a pour point

### 3.3 Terrain Factor
- **Slope** — derived from the filled DEM (percent rise), highlighting terrain
  prone to rapid runoff generation

### 3.4 Stream Proximity Factor
- **Distance to Stream** — Euclidean distance from the derived stream network,
  used as a proxy for flood exposure

### 3.5 Reclassification
Each input raster (Flow Accumulation, Slope, Distance to Stream) is reclassified
onto a common **1–5 suitability scale**, where 5 represents the highest flash
flood susceptibility.

### 3.6 Weighted Overlay
The reclassified layers are combined using **Weighted Overlay**:

| Factor              | Weight |
|----------------------|--------|
| Distance to Stream    | 40%    |
| Flow Accumulation     | 25%    |
| Slope                 | 35%    |

The output is a single flash flood susceptibility raster (1–5 scale).

## 4. Validation

The resulting susceptibility map is cross-checked against reference flood hazard
data from **LUBW** (Landesanstalt für Umwelt Baden-Württemberg) to assess
agreement between the model output and officially recognized flood-prone areas.

## 5. Software & Tools

- Esri ArcGIS Pro — Spatial Analyst extension
- QGIS
- Tools used: Fill, Flow Direction, Flow Accumulation, Watershed, Slope,
  Euclidean Distance, Reclassify, Weighted Overlay
