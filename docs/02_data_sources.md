# Data Sources

## Digital Elevation Model — DGM1

| Property        | Detail |
|------------------|--------|
| Dataset          | Digitales Geländemodell 1 m (DGM1) |
| Resolution       | 1 m grid spacing |
| Coverage         | Wolfach sub-catchment, Ortenaukreis, Baden-Württemberg |
| Provider         | LGL-BW (Landesamt für Geoinformation und Landentwicklung Baden-Württemberg) |
| Access portal    | https://opengeodata.lgl-bw.de/ |
| License          | Datenlizenz Deutschland – Namensnennung – Version 2.0 (dl-de/by-2-0) |
| License text     | https://www.govdata.de/dl-de/by-2-0 |

### Attribution

> Datenquelle: LGL-BW (2026), Digitales Geländemodell 1 m (DGM1),
> Datenlizenz Deutschland – Namensnennung – Version 2.0, www.lgl-bw.de

## Reference / Validation Data

| Property        | Detail |
|------------------|--------|
| Dataset          | Flood hazard reference data |
| Provider         | LUBW (Landesanstalt für Umwelt Baden-Württemberg) |
| Purpose          | Validation of the derived susceptibility map |
| Access           | https://rips-metadaten.lubw.de/ |

## Notes on Repository Data Handling

Raw DGM1 tiles are **not stored in this repository** due to file size limits on
GitHub. Only derived, lightweight outputs (e.g. exported map images, reclassified
layers where feasible) are versioned. To reproduce the analysis:

1. Download the relevant DGM1 tiles for the Wolfach sub-catchment from the
   [LGL-BW Open GeoData portal](https://opengeodata.lgl-bw.de/)
2. Place the tiles in `data/raw/`
3. Follow the steps in [`01_methodology.md`](01_methodology.md)

All use of LGL-BW data in this project complies with the terms of the
Datenlizenz Deutschland – Namensnennung – Version 2.0 license, which permits
both non-commercial and commercial use subject to attribution.
