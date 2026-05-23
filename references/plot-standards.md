# Plot Standards

Read before writing or changing plot code. The rendered PNG, not code parameters, is ground truth.

## Colormaps and Levels

`import cmaps` and explicit discrete `levels` for filled fields. Never `jet`, `rainbow`, `viridis`, `MPL_jet`, `MPL_rainbow`, `NCV_jet`.

| Context | Recommended `cmaps` | Rule |
|---|---|---|
| Temperature | `temp_19lev`, `hotcolr_19lev` | Sequential |
| Temperature anomaly | `temp_diff_18lev`, `BlueWhiteOrangeRed`, `BlWhRe` | Diverging, center 0 |
| Precipitation | `precip_11lev`, `precip3_16lev`, `CBR_wet` | White/light = dry |
| Precipitation anomaly | `precip_diff_12lev`, `CBR_drywet`, `GMT_drywet` | Brown=dry, green=wet |
| Wind speed | `wind_17lev` | Sequential |
| SST | `temp_19lev`, `cmocean_thermal` | Sequential |
| SST anomaly | `GHRSST_anomaly`, `MPL_sstanom` | Diverging, center 0 |
| Geopotential / SLP | `BlAqGrYeOrRe`, `BlAqGrYeOrReVi200`, `MPL_YlOrRd` | Sequential |
| Humidity / moisture | `MPL_BuGn`, `MPL_YlGnBu` | Sequential |
| Vorticity / divergence / omega | `BlWhRe`, `NCV_blu_red`, `hotcold_18lev` | Diverging, center 0 |
| Correlation / regression | `NCV_blu_red`, `BlueWhiteOrangeRed`, `MPL_RdBu_r` | Symmetric when signed |
| Topography | `GMT_topo`, `topo_15lev` | Terrain |
| Generic absolute | `BlAqGrYeOrReVi200`, `MPL_YlOrRd` | Sequential |
| Generic anomaly | `BlueWhiteOrangeRed`, `BlWhRe`, `NCV_blu_red` | Diverging, center 0 |

### Diverging anomaly scale

MUST:

- Center at zero.
- Include zero in ticks.
- Use matched positive and negative limits unless justified.
- Use `TwoSlopeNorm(vcenter=0)` or explicit symmetric `BoundaryNorm`.

### Sequential scale

MUST:

- Use monotonic lightness when possible.
- Avoid using categorical colors for continuous variables.
- Use physically meaningful endpoints and ticks.

Set scientifically interpretable levels from domain knowledge. Don't auto-scale per panel. If the signal is hidden or saturated, inspect summary ranges, then set common readable levels.

### Colorbar design

- Horizontal colorbars work well below maps and compact multi-panel groups.
- Vertical colorbars work well for standalone maps or right-side shared legends.
- Use short unit labels: `SST anomaly (°C)`, `Wind speed (m s–1)`, `Precipitation (mm day–1)`.
- Avoid colorbars longer than the panel group they describe.
- Use the same ticks and limits for comparable panels.

### Forbidden

- `jet`, `rainbow`, `nipy_spectral`, and saturated hue wheels for quantitative fields.
- Red–green-only categorical encodings.
- Diverging maps not centered on zero for anomaly fields.
- Per-panel autoscaled colorbars for a panel set that should be compared.
- Colorbars without units or with too many tick labels.

## Projections

| Region | Projection |
|---|---|
| Global | `PlateCarree(central_longitude=180)` or `Robinson` |
| Pacific-centered | `PlateCarree(central_longitude=180)` |
| Tropics | `Mercator` |
| Mid-latitude regional | `PlateCarree` (default) or `LambertConformal` |
| Polar | `NorthPolarStereo` / `SouthPolarStereo` |

- For global fields, add a cyclic point or roll coordinates to avoid a dateline seam.

## Layout

Use Arial for all figure text:

```python
plt.rcParams["font.family"] = "Arial"
```

Units must be written exponentially, e.g. `W m–2`, `kg m–2 s–1`, `m s–1`. Do not use slash notation such as `W/m^2`.

Starting figure sizes:

| Layout | Size |
|---|---|
| Single map | `(10, 6)`–`(12, 8)` |
| 2-panel side-by-side | `(14, 5)`–`(16, 6)` |
| 2-panel stacked | `(10, 10)`–`(12, 12)` |
| 4-panel | `(12, 10)`–`(14, 12)` |
| 6-panel | `(16, 10)`–`(18, 12)` |

- `constrained_layout=True` for simple grids; `GridSpec` / `subplot_mosaic` for 3+ panels, shared colorbars, or mixed content.
- Shared variable + range → one shared colorbar; size with `shrink`, `aspect`, or a dedicated narrow `GridSpec` row/column.
- Minimum readable font 7 pt; prefer 8–11 pt for ticks, labels, legends, colorbars.
- For 3+ map panels, prefer vertical stacking or 2×2 / 3×2 over a cramped 1×N row.

### Panel labels

- Use lower-case bold upright labels: `a`, `b`, `c`, ...
- Place labels at the upper-left of each panel or slightly outside the axes.
- Align labels across rows/columns.
- Do not include punctuation after panel letters.

### Titles

- Use short, descriptive panel titles.
- Avoid redundant titles when rows/columns already have labels.
- For experiments, use compact labels like `Historical`, `SSP5-8.5`, `El Niño composite`, `La Niña composite`.

### Spacing

- Keep whitespace minimal but not cramped.
- Use `constrained_layout=True` or `GridSpec` with explicit ratios.
- For Cartopy maps, check layout after adding colorbars because GeoAxes often produce unexpected spacing.
- Do not let colorbars determine the whole figure geometry.

### Shared elements

- Shared colorbar: use for the same variable and scale.
- Shared legend: use for repeated categories across panels.
- Shared axis labels: use when a grid of statistical plots repeats the same axes.
- Shared annotation: place in margin only if it clarifies the entire figure.

## Maps

- Coastlines by default. Do not draw national/provincial borders by default; add them only when scientifically or geographically necessary.
- Indicate latitude and longitude on every map with ticks or gridline labels.
- Gridlines: `draw_labels=True`; disable top/right labels; tick font 8–10 
- Use dashed red/blue boxes for study regions; linewidth `1.2–2.0` and high zorder.

## Tibetan Plateau Masking

For plots covering ~25°N–40°N, 70°E–105°E, mask low-level data over Tibet.

| Level | Action |
|---|---|
| 1000 / 925 / 850 hPa | Mask required |
| 700 hPa | Mask recommended |
| ≥ 500 hPa | No mask |

Applies to wind, height, temperature, humidity, vorticity, divergence — any pressure-level field at or below 850 hPa.

```python
import geopandas as gpd
import cartopy.crs as ccrs

tibet = gpd.read_file("~/code/data/map/Tibet/Tibet.shp")
ax.add_geometries(
    tibet.geometry, crs=ccrs.PlateCarree(),
    facecolor="lightgrey", edgecolor="grey", linewidth=0.5, zorder=5,
)
```

Place the mask after filled/vector layers, before final borders/annotations. For quiver, also mask `u`/`v` over Tibet or place the polygon above arrows.

## Vectors and Contours

Goal: arrows distinguishable, field structure visible.

- Use muted grey arrows (`#555555`) with width `0.0025–0.004`; avoid black arrow fields unless sparse.
- Skip factor by resolution: 2.5° → 1–2; 1° → 3–5; 0.25° → 8–15. Larger domains need more skipping.
- Reference magnitude: rounded value near median speed, not maximum.
- Place quiver key outside/top-right; top-left is reserved for panel letters.
- For dense or warped vector fields, use Cartopy `regrid_shape=20` as a first guess.
- Overlay contours only when they add physical interpretation; use thin dark grey/black lines and labels that do not dominate.

```python
Q = ax.quiver(lon[::n], lat[::n], u[::n, ::n], v[::n, ::n],
              transform=ccrs.PlateCarree(), color="#555555",
              scale=200, width=0.003, zorder=3)
ax.quiverkey(Q, 0.90, 1.046, 10, "10 m s–1", labelpos="E",
             coordinates="axes", fontproperties={"size": 9}, labelsep=0.05)
```

## Significance

Make significance visible without obscuring the field.

- Hatching: sparse `"..."`; avoid dense `"/////"` or `"xxxxx"`.
- Scatter: choose marker size against the rendered PNG; start around 0.3–1.0 with alpha 0.4–0.6.
- Use dark gray/black with alpha 0.35–0.7; avoid saturated stippling colors.
- Subsample every 2–4 grid points when needed. High-resolution grids need stronger subsampling to avoid black patches while preserving significant-region structure.

```python
ax.contourf(lon, lat, p_value, levels=[0, 0.05, 1],
            hatches=["...", None], colors="none",
            transform=ccrs.PlateCarree())
```

## Gridlines and tick labels

- Use sparse, readable lon/lat labels.
- Avoid labeling every gridline; typical intervals are 30°/60° globally and 5°/10°/20° regionally.
- Do not use both dense tick labels and dense gridlines.
- For multi-panel maps, label only the left column and bottom row when possible.

## Contour overlays

- Use contour overlays to clarify gradients, zero crossings, or physically meaningful thresholds.
- Label only selected contours. Too many contour labels degrade readability.
- Zero contour: dark neutral or black, 0.8–1.0 pt.
- Secondary contours: 0.4–0.6 pt, moderate alpha.

## Other Figure Types

### Time series

- Use thin lines with restrained markers.
- Use zero/reference lines for anomaly indices; red/orange for heatwave/positive index, light blue bars for precipitation/paired index when appropriate.
- Show uncertainty using shaded confidence bands, ensemble spread, or error bars when available.
- Use seasonal/year tick locators deliberately; avoid overcrowded date labels.
- Use direct labels for 2–4 lines when clearer than a legend.
- Annotate correlation, p markers, and highlighted years sparingly.

### Seasonal cycle

- Use month labels or compact month initials.
- Show climatology and experiments with consistent line styles.
- Use shaded spread if comparing ensembles.
- Keep y-axis units explicit.

### Scatter and regression

- Use semi-transparent small markers; alpha for dense data.
- Add regression line and confidence interval only when statistically justified.
- Report `r`, `R²`, slope, or p-value in a small annotation if central to the result.
- Do not use oversized bubble markers unless size is a meaningful third variable.

### Cross-section / profile

- Pressure axis inverted (1000 bottom → 100 top). Use symlog only when scientifically justified.
- Fill topography/orography in grey.

### Hovmöller

- Same colormap rules as maps; time on one axis, lat/lon on the other.

### Heatmaps and correlation matrices

- Use diverging scales for correlations and signed differences.
- Center correlation scales at zero and set limits to `[-1, 1]` unless comparing a restricted range deliberately.
- Use square cells for matrices when possible.
- Rotate labels only as much as needed.
- Mark significance with small dots, outlines, or hatches rather than overprinting numbers in every cell.

### Box, violin, and bar plots

- Prefer showing the distribution when sample size is small or moderate: jittered points, box + points, or violin + median.
- Use bars for aggregated quantities only when the aggregate is the message.
- Show uncertainty: standard error, confidence interval, interquartile range, or ensemble spread.
- Do not use 3D bars or gradient-filled bars.

### EOF / PCA diagnostics

- Spatial modes: map panels with diverging colormap centered at zero.
- PCs: normalized time series with zero line.
- Explained variance: include percent in title or label.
- Keep sign conventions explicit if modes are flipped.

### Taylor diagrams and skill summaries

- Correlation angular axis, std-ratio radial axis, reference at `(1, 1)`.
- Keep markers readable; use direct labels or a compact legend.
- Avoid too many model markers in one diagram. Split into groups if necessary.
- Use muted categorical colors and marker shapes.

### Volcano plots and generic scatter summaries

Volcano plots are not core atmos/ocean diagnostics but may appear in interdisciplinary work.

- Use muted nonsignificant points and one or two accent colors for significant groups.
- Label only top-priority points.
- Add threshold lines and explain them in the legend or caption.
- Avoid saturated red/green-only up/down encoding.

## Export

```python
fig.savefig("path.png", dpi=600, bbox_inches="tight", pad_inches=0.05)
plt.close(fig)
```
