# Plot Standards

Use this file for any figure rendering or `RR` step.

## 1. Core defaults

- Publication-quality, professional, clean, informative.
- No main title. No source/process stamps (e.g., `ERA5 monthly mean`) inside the figure — put them in the caption or README.
- Subfigure labels `(a)`, `(b)`, `(c)` with brief subtitles, placed just outside the upper-left corner of each subfigure.
- No overlapping or clipped text, ticks, legends, colorbars, or panel labels.
- White background, restrained styling. No 3D effects, shadows, gradients, or patterned fills.
- Font: Arial. Size: 10–12 pt single-panel, 8–10 pt multi-panel, never below 7 pt.
- English in-figure text by default unless the user or project requires otherwise.
- If a variable uses `contourf`, do not add contour lines unless they convey additional information (e.g., geopotential height contours over a temperature fill).
- Every colorbar must include units. Use scientifically meaningful ticks.

## 2. Colormap

All colormaps must come from `cmaps` with discrete levels. Choose range and interval from the variable and science.

| Variable / context | Style | Recommended `cmaps` | Notes |
|---|---|---|---|
| Temperature (absolute) | Sequential warm | `temp_19lev`, `hotcolr_19lev` | |
| Temperature (anomaly) | Diverging, white at 0 | `temp_diff_18lev`, `BlueWhiteOrangeRed`, `BlWhRe` | Symmetric range |
| Precipitation (absolute) | Sequential wet | `precip_11lev`, `precip3_16lev`, `CBR_wet` | White for dry end |
| Precipitation (anomaly) | Diverging | `precip_diff_12lev`, `CBR_drywet`, `GMT_drywet` | Brown=dry, green=wet |
| Wind speed | Sequential | `wind_17lev` | |
| SST (absolute) | Sequential warm | `temp_19lev`, `cmocean_thermal` | |
| SST (anomaly) | Diverging | `GHRSST_anomaly`, `MPL_sstanom` | Center at 0 |
| Geopotential height | Sequential multi-hue | `BlAqGrYeOrRe`, `BlAqGrYeOrReVi200` | |
| Humidity / moisture | Sequential blue-green | `MPL_BuGn`, `MPL_YlGnBu` | |
| OLR / radiation | Sequential warm | `sunshine_9lev`, `MPL_YlOrRd` | |
| Vorticity / divergence | Diverging | `BlWhRe`, `NCV_blu_red`, `hotcold_18lev` | Center at 0 |
| Correlation / regression | Diverging | `NCV_blu_red`, `BlueWhiteOrangeRed`, `MPL_RdBu_r` | Symmetric range |
| Topography | Terrain | `GMT_topo`, `topo_15lev` | |
| Generic anomaly / difference | Diverging, white at 0 | `BlueWhiteOrangeRed`, `BlWhRe`, `NCV_blu_red` | Always center at 0 |
| Generic absolute field | Sequential | `BlAqGrYeOrReVi200`, `MPL_YlOrRd` | |

Avoid `MPL_jet`, `MPL_rainbow`, `MPL_viridis`, `NCV_jet`. Comparable panels share the same colormap and range unless units differ.

## 3. Maps

### 3.1 Setup

- Coastlines by default; national borders only if requested.
- Show lon/lat; handle 0/360 ↔ −180/180 transitions.

### 3.2 Projection

| Region | Projection |
|---|---|
| Global | `PlateCarree` or `Robinson` |
| Tropics | `PlateCarree` or `Mercator` |
| Mid-latitude regional | `LambertConformal` (set `central_longitude`, `standard_parallels`) |
| Polar | `NorthPolarStereo` / `SouthPolarStereo` |
| Pacific-centered | `PlateCarree(central_longitude=180)` |

### 3.3 Overlays

- Vector fields: grid skip every 3–5 pts for 1°, 2–3 pts for 2.5°. Place `quiver key` with units in an uncluttered corner.
- Significance stippling: use `.` markers at intervals, or sparse hatching with alpha < 0.5. Do not obscure the main signal.

## 4. Other figure types

- Pastel palette by default when colors beyond black are needed.
- **Time series**: clear line styles, legend, human-readable dates, zero line for anomalies.
- **Vertical cross-sections**: pressure (hPa) y-axis inverted (1000 bottom → 100 top), latitude/longitude x-axis.
- **Hovmöller**: time on one axis, space on the other, same colormap rules as maps.
- **Scatter / regression**: regression line, R², p-value, transparency for dense clouds.

## 5. Layout and export

- Balanced whitespace; `constrained_layout=True` or explicit `gridspec` for multi-panel.
- Share legends/colorbars across comparable panels when appropriate.
- Export `PNG` (600 dpi) and `SVG`.

## 6. Quick reject checklist

Revise immediately if any item is true:
- Main title or source/process stamp in figure
- Text, ticks, legends, colorbars overlap or clip; font below 7 pt
- Missing axis labels, units, colorbar, or `(a)`/`(b)` labels on multi-panel
- Colormap not from `cmaps` or not using discrete levels
- Anomaly/difference field not using diverging colormap centered at 0
- Comparable panels with misleadingly different color scales
- Wrong projection for region; missing coastlines or lon/lat
- Vector density inappropriate; reference arrow missing or unlabeled
- Significance stippling obscures the main signal
- Redundant contour lines on `contourf` without added information
