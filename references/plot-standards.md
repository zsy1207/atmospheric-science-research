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
| Sea level pressure | Sequential | `MPL_YlOrRd`, `BlAqGrYeOrRe` | Low=warm, high=cool |
| Cloud cover | Sequential grey-blue | `MPL_Greys`, `MPL_GnBu` | White for clear |
| Soil moisture | Sequential blue | `MPL_YlGnBu`, `MPL_BuGn` | |
| Snow / sea ice | Sequential cold | `MPL_Blues`, `NCV_bright` | White for ice-free |
| Vertical velocity (omega) | Diverging | `BlWhRe`, `NCV_blu_red` | Negative=upward convention; center at 0 |
| Relative humidity | Sequential green-blue | `MPL_YlGnBu`, `MPL_BuGn` | |
| Aerosol optical depth | Sequential warm | `MPL_YlOrRd`, `MPL_hot_r` | |
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
| Tropics | `Mercator` |
| Mid-latitude regional | `LambertConformal` (set `central_longitude`, `standard_parallels`) |
| Polar | `NorthPolarStereo` / `SouthPolarStereo` |
| Pacific-centered | `PlateCarree(central_longitude=180)` |

### 3.3 Overlays

- Vector fields: grid skip every 3–5 pts for 1°, 2–3 pts for 2.5°. Place `quiver key` with units in an uncluttered corner.
- Significance stippling: sparse markers or hatching, never obscuring the fill signal.

```python
# Stippling pattern — use after contourf
ax.contourf(lon, lat, p_value, levels=[0, 0.05, 1],
            hatches=["...", None], colors="none",
            transform=ccrs.PlateCarree())
# Or scatter-based for finer control
sig = p_value < 0.05
lon2d, lat2d = np.meshgrid(lon, lat)
ax.scatter(lon2d[sig][::3], lat2d[sig][::3], s=0.5, c="k", alpha=0.5,
           transform=ccrs.PlateCarree())
```

## 4. Other figure types

- Pastel palette by default when colors beyond black are needed.
- **Time series**: clear line styles, legend, human-readable dates, zero line for anomalies.
- **Vertical cross-sections**: pressure (hPa) y-axis inverted (1000 bottom → 100 top), latitude/longitude x-axis.
- **Hovmöller**: time on one axis, space on the other, same colormap rules as maps.
- **Scatter / regression**: regression line, R², p-value, transparency for dense clouds.

## 5. Layout and export

### 5.1 Figure sizing

| Layout | `figsize` (inches) |
|---|---|
| Single map | (10, 6) – (12, 8) |
| Single time series | (10, 4) – (12, 5) |
| 2-panel side-by-side maps | (14, 5) – (16, 6) |
| 2-panel stacked maps | (10, 10) – (12, 12) |
| 4-panel grid (2×2) | (12, 10) – (14, 12) |
| 6-panel grid (2×3 or 3×2) | (16, 10) – (18, 12) |
| Map + time series (top/bottom) | (10, 8) – (12, 10) |

Adjust proportionally for the actual aspect ratio. Prefer wider figures for maps, taller for vertical cross-sections.

### 5.2 Multi-panel strategy

- Use `fig, axes = plt.subplots(nrows, ncols, figsize=..., subplot_kw={"projection": ...}, constrained_layout=True)`.
- When panels share the same variable and range, use one shared colorbar placed at the bottom or right of the figure group.
- For `gridspec_kw`, use `hspace` / `wspace` ≈ 0.05–0.15 with `constrained_layout`, or explicit `GridSpec` ratios for asymmetric layouts (e.g., map + colorbar + time series).
- Panel labels `(a)`, `(b)`, … go at upper-left of each axes, slightly outside the frame: `ax.text(-0.02, 1.03, "(a)", transform=ax.transAxes, fontsize=11, fontweight="bold", va="bottom", ha="right")`.

### 5.3 Export

- Export `PNG` (600 dpi) and `SVG`.
- Use `bbox_inches="tight"` to avoid clipped labels.
- Verify exported files are non-empty.

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
- White gaps or seams in filled contour at dateline or plot boundary
- Legend outside visible area or occluding data
- Inverted y-axis missing on pressure-level cross-sections
