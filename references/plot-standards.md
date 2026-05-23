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

- Center at zero; include zero in ticks; use matched positive/negative limits unless justified.
- Use `TwoSlopeNorm(vcenter=0)` or an explicit symmetric `BoundaryNorm`.
- Pass `extend="both"` to `contourf` / `colorbar` so out-of-range values render as extension triangles, not silently clipped fill.
- For comparable panels, build **one shared `BoundaryNorm`** once and reuse it across every panel — never let panels recompute their own norm.

### Sequential scale

MUST:

- Use monotonic lightness; avoid categorical colors for continuous variables.
- Use physically meaningful endpoints and ticks.
- Pass `extend="both"` (or `"max"` / `"min"` when one tail is physically closed, e.g. precipitation ≥ 0) whenever values can exceed `levels`.
- For comparable panels, share a single `BoundaryNorm` and `levels` array across all panels.

Set scientifically interpretable levels from domain knowledge. Never auto-scale per panel. If the signal is hidden or saturated, inspect summary ranges, then set common readable levels.

### Colorbar design

- Horizontal colorbars below maps and compact multi-panel groups; vertical for standalone maps or right-side shared legends.
- Short unit labels: `SST anomaly (°C)`, `Wind speed (m s−1)`, `Precipitation (mm day−1)`.
- Never longer than the panel group it describes.
- Same ticks and limits across comparable panels.

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
  ```python
  from cartopy.util import add_cyclic_point
  # da: xarray.DataArray with dims (lat, lon)
  cyclic_data, cyclic_lon = add_cyclic_point(da.values, coord=da["lon"].values, axis=-1)
  ax.contourf(cyclic_lon, da["lat"].values, cyclic_data, transform=ccrs.PlateCarree())
  ```
- **Pass `transform=ccrs.PlateCarree()`** (or the actual data CRS) to every Cartopy plot call: `contourf`, `contour`, `pcolormesh`, `quiver`, `streamplot`, `scatter`, `add_geometries`. Data CRS and display projection are independent; omitting `transform` silently treats data coordinates as projected meters and produces a wrong map.

## Layout

Use Arial for all figure text:

```python
plt.rcParams["font.family"] = "Arial"
```

Write units exponentially: `W m−2`, `kg m−2 s−1`, `m s−1`. Never slash notation (`W/m^2`).

Starting figure sizes — AGU/Wiley standard widths (1-column 95 mm = 3.74 in, 1.5-column 140 mm = 5.51 in, 2-column 190 mm = 7.48 in; max figure 190 × 230 mm = 7.48 × 9.06 in):

| Layout | `figsize` (in) | Column |
|---|---|---|
| Single map, compact | `(3.74, 2.6)`–`(3.74, 3.5)` | 1 |
| Single map, full width | `(7.48, 4.5)`–`(7.48, 6.0)` | 2 |
| 2-panel side-by-side | `(7.48, 3.0)`–`(7.48, 3.8)` | 2 |
| 2-panel stacked | `(3.74, 4.8)`–`(3.74, 5.6)` | 1 |
| 4-panel (2×2) | `(7.48, 5.0)`–`(7.48, 6.5)` | 2 |
| 6-panel (3×2) | `(7.48, 6.5)`–`(7.48, 8.0)` | 2 |
| Hard ceiling | `(7.48, 9.06)` | — |

- `constrained_layout=True` for simple grids; `GridSpec` / `subplot_mosaic` for 3+ panels, shared colorbars, or mixed content.
- Shared variable + range → one shared colorbar; size with `shrink`, `aspect`, or a dedicated narrow `GridSpec` row/column.
- Minimum readable font 7 pt; prefer 8–11 pt for ticks, labels, legends, colorbars.
- For 3+ map panels, prefer vertical stacking or 2×2 / 3×2 over a cramped 1×N row.

### Panel labels and descriptions

- Lower-case bold upright labels: `a`, `b`, `c`, ...; aligned across rows/columns; no punctuation after the letter.
- Panel letters at the upper-left of each panel (or slightly outside the axes); descriptions go upper-right or as row/column labels.
- Compact descriptors only where they clarify: `Historical`, `SSP5-8.5`, `850 hPa wind`, `El Niño composite`.
- No figure-level `suptitle`, default centered axis titles, captions, or data stamps.

### Spacing

- Whitespace minimal but not cramped.
- `constrained_layout=True` or `GridSpec` with explicit ratios.
- For Cartopy maps, recheck layout after adding colorbars — GeoAxes often produce unexpected spacing.
- Colorbars never determine the whole figure geometry.

### Shared elements

- Shared colorbar for the same variable and scale.
- Shared legend for repeated categories across panels.
- Shared axis labels for a grid of statistical plots with repeating axes.
- Shared annotation in margin only if it clarifies the entire figure.

## Maps

- Coastlines by default. No national/provincial borders unless scientifically/geographically necessary.
- Indicate latitude/longitude on every map via ticks or gridline labels.
- Gridlines: `draw_labels=True`; disable top/right labels; tick font 8–10 pt.
- Dashed red/blue boxes for study regions; linewidth `1.2–2.0`, high zorder.

## Tibetan Plateau Masking

For plots covering ~25°N–40°N, 70°E–105°E, mask low-level data over Tibet.

| Level | Action |
|---|---|
| 1000 / 925 / 850 hPa | Mask required |
| 700 hPa | Mask recommended |
| ≥ 500 hPa | No mask |

Applies to wind, height, temperature, humidity, vorticity, divergence — any pressure-level field at or below 850 hPa.

### 1. Physical Masking (Computation Phase)
Keep grid points where `ps ≥ P_level`; mask the rest (they lie below the topography).

```python
# data at P_level (hPa); ps in hPa. ERA5 raw `sp` is in Pa — convert first,
# else `ps >= 850` silently masks the entire field.
data_masked = data.where(ps >= 850)   # generalize: ps >= P_level
```

### 2. Visual Masking (Plotting Phase)
Overlay a grey polygon to present the topography boundary cleanly:

```python
import geopandas as gpd
import cartopy.crs as ccrs
import os

tibet_shp = os.path.expanduser("~/code/data/map/Tibet/Tibet.shp")
if os.path.exists(tibet_shp):
    tibet = gpd.read_file(tibet_shp)
    ax.add_geometries(
        tibet.geometry, crs=ccrs.PlateCarree(),
        facecolor="lightgrey", edgecolor="grey", linewidth=0.5, zorder=5,
    )
```

Place the visual mask after filled/vector layers and before final borders/annotations. For quiver, also mask `u`/`v` over Tibet or place the polygon above arrows.

## Vectors and Contours

Goal: arrows distinguishable, field structure visible.

- Muted grey arrows (`#555555`), width `0.0025–0.004`; avoid black arrow fields unless sparse.
- Skip factor by resolution: 2.5° → 1–2; 1° → 3–5; 0.25° → 8–15. Larger domains need more skipping.
- Reference magnitude: rounded value near the median speed, not the maximum.
- Quiver key outside/top-right; top-left is reserved for panel letters.
- For dense or warped fields, try Cartopy `regrid_shape=20` as a first guess.
- Overlay contours only when they add physical interpretation; thin dark grey/black lines, labels that do not dominate.

```python
Q = ax.quiver(lon[::n], lat[::n], u[::n, ::n], v[::n, ::n],
              transform=ccrs.PlateCarree(), color="#555555",
              scale=200, width=0.003, zorder=3)
ax.quiverkey(Q, 0.90, 1.046, 10, "10 m s−1", labelpos="E",
             coordinates="axes", fontproperties={"size": 9}, labelsep=0.05)
```

## Significance

Make significance visible without obscuring the field.

- Hatching: sparse `"..."`; avoid dense `"/////"` or `"xxxxx"`.
- Scatter: tune marker size against the rendered PNG; start `0.3–1.0` with alpha `0.4–0.6`.
- Dark grey/black with alpha `0.35–0.7`; avoid saturated stippling colors.
- Subsample every 2–4 grid points; high-resolution grids need stronger subsampling to avoid black patches while preserving significant-region structure.

```python
ax.contourf(lon, lat, p_value, levels=[0, 0.05, 1],
            hatches=["...", None], colors="none",
            transform=ccrs.PlateCarree())
```

## Gridlines and tick labels

- Sparse, readable lon/lat labels; typical intervals 30°/60° globally and 5°/10°/20° regionally.
- Never combine dense tick labels with dense gridlines.
- Multi-panel maps: label only the left column and bottom row when possible.

## Contour overlays

- Use to clarify gradients, zero crossings, or physically meaningful thresholds.
- Label selected contours only — too many labels degrade readability.
- Zero contour: dark neutral or black, `0.8–1.0` pt.
- Secondary contours: `0.4–0.6` pt, moderate alpha.

## Other Figure Types

### Time series

- Thin lines with restrained markers; zero/reference lines for anomaly indices (red/orange for heatwave or positive index, light blue bars for precipitation or paired index).
- Show uncertainty via shaded confidence bands, ensemble spread, or error bars.
- Seasonal/year tick locators deliberate; no overcrowded date labels.
- Direct labels for 2–4 lines when clearer than a legend.
- Annotate correlation, p markers, and highlighted years sparingly.

### Seasonal cycle

- Month labels or compact month initials.
- Consistent line styles between climatology and experiments; shaded spread for ensembles.
- Explicit y-axis units.

### Scatter and regression

- Semi-transparent small markers; lower alpha for dense data.
- Regression line and confidence interval only when statistically justified.
- Report `r`, `R²`, slope, or p-value in a small annotation if central to the result.
- No oversized bubble markers unless size is a meaningful third variable.

### Cross-section / profile

- Pressure axis inverted (1000 bottom → 100 top); symlog only when scientifically justified.
- Fill topography/orography in grey.

### Hovmöller

- Same colormap rules as maps; time on one axis, lat/lon on the other.

### Heatmaps and correlation matrices

- Diverging scales for correlations and signed differences; center at zero with limits `[-1, 1]` unless deliberately restricted.
- Square cells when possible; rotate labels only as needed.
- Mark significance with small dots, outlines, or hatches — never overprint numbers in every cell.

### Box, violin, and bar plots

- Prefer the distribution for small/moderate samples: jittered points, box + points, or violin + median.
- Bars only when the aggregate is the message.
- Show uncertainty: standard error, CI, IQR, or ensemble spread.
- No 3D bars or gradient-filled bars.

### EOF / PCA diagnostics

- Spatial modes: map panels with diverging colormap centered at zero.
- PCs: normalized time series with zero line.
- Explained variance: percent in title or label.
- Explicit sign convention if modes are flipped.

### Taylor diagrams and skill summaries

- Correlation angular axis, std-ratio radial axis, reference at `(1, 1)`.
- Readable markers; direct labels or compact legend.
- Split into groups when many model markers.
- Muted categorical colors and marker shapes.

### Volcano plots and generic scatter summaries

Not core atmos/ocean diagnostics but may appear in interdisciplinary work.

- Muted nonsignificant points; one or two accent colors for significant groups.
- Label only top-priority points.
- Add threshold lines with legend/caption explanation.
- No saturated red/green-only up/down encoding.

## Export

```python
fig.savefig("path.png", dpi=600, bbox_inches="tight", pad_inches=0.05)
plt.close(fig)
```

`bbox_inches="tight"` may shrink the final canvas by up to ±5% from the `figsize` in the AGU width table; treat the listed sizes as targets, not exact constraints.
