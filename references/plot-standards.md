# Plot Standards

> [Colormaps](#colormap-rules) · [Projections](#projections) · [Figure Sizing](#figure-sizing) · [Multi-panel](#multi-panel-layout) · [Maps](#maps) · [Vectors](#vectors-wind-currents) · [Stippling](#significance-stippling) · [Other Figures](#other-figure-types) · [Export](#export) · [Quick Reject](#quick-reject-checklist)

## Colormap Rules

Colormaps from `import cmaps` with discrete `levels`. Avoid `MPL_jet`, `MPL_rainbow`, `MPL_viridis`, `NCV_jet` — these create perceptual artifacts that distort data interpretation (e.g., false boundaries where none exist).

**Principles**:
- Absolute fields → sequential colormap (low=light, high=dark)
- Anomaly / difference → diverging colormap, symmetric around 0 (center=white or neutral)
- Comparable panels share the same colormap and level range unless units differ

| Variable / context | Style | Recommended `cmaps` | Notes |
|---|---|---|---|
| Temperature absolute | Sequential warm | `temp_19lev`, `hotcolr_19lev` | |
| Temperature anomaly | Diverging | `temp_diff_18lev`, `BlueWhiteOrangeRed`, `BlWhRe` | Symmetric, center 0 |
| Precipitation absolute | Sequential wet | `precip_11lev`, `precip3_16lev`, `CBR_wet` | White=dry end |
| Precipitation anomaly | Diverging | `precip_diff_12lev`, `CBR_drywet`, `GMT_drywet` | Brown=dry, green=wet |
| Wind speed | Sequential | `wind_17lev` | |
| SST absolute | Sequential warm | `temp_19lev`, `cmocean_thermal` | |
| SST anomaly | Diverging | `GHRSST_anomaly`, `MPL_sstanom` | Center 0 |
| Geopotential height | Sequential multi-hue | `BlAqGrYeOrRe`, `BlAqGrYeOrReVi200` | |
| Humidity / moisture | Sequential blue-green | `MPL_BuGn`, `MPL_YlGnBu` | |
| OLR / radiation | Sequential warm | `sunshine_9lev`, `MPL_YlOrRd` | |
| Vorticity / divergence | Diverging | `BlWhRe`, `NCV_blu_red`, `hotcold_18lev` | Center 0 |
| Correlation / regression | Diverging | `NCV_blu_red`, `BlueWhiteOrangeRed`, `MPL_RdBu_r` | Symmetric |
| Topography | Terrain | `GMT_topo`, `topo_15lev` | |
| Sea level pressure | Sequential | `MPL_YlOrRd`, `BlAqGrYeOrRe` | |
| Cloud cover | Grey-blue | `MPL_Greys`, `MPL_GnBu` | White=clear |
| Soil moisture | Sequential blue | `MPL_YlGnBu`, `MPL_BuGn` | |
| Snow / sea ice | Sequential cold | `MPL_Blues`, `NCV_bright` | White=ice-free |
| Vertical velocity | Diverging | `BlWhRe`, `NCV_blu_red` | Neg=upward, center 0 |
| Relative humidity | Sequential green-blue | `MPL_YlGnBu`, `MPL_BuGn` | |
| Aerosol optical depth | Sequential warm | `MPL_YlOrRd`, `MPL_hot_r` | |
| Generic anomaly | Diverging | `BlueWhiteOrangeRed`, `BlWhRe`, `NCV_blu_red` | Center 0 |
| Generic absolute | Sequential | `BlAqGrYeOrReVi200`, `MPL_YlOrRd` | |

## Projections

Choose projections that minimize distortion for the study region.

| Region | Projection |
|---|---|
| Global | `PlateCarree` or `Robinson` |
| Tropics | `Mercator` |
| Mid-latitude regional | `PlateCarree` or `LambertConformal` (set `central_longitude`, `standard_parallels`) |
| Polar | `NorthPolarStereo` / `SouthPolarStereo` |
| Pacific-centered | `PlateCarree(central_longitude=180)` |

## Figure Sizing

Starting-point sizes — **always verify proportions in the rendered PNG and adjust iteratively**, because the right figsize depends on content density, colorbar placement, annotations, and output medium. Code-level sizing alone is unreliable.

| Layout | Starting `figsize` (inches) | Notes |
|---|---|---|
| Single map | (10, 6)–(12, 8) | |
| Single time series | (10, 4)–(12, 5) | |
| 2-panel side-by-side | (14, 5)–(16, 6) | |
| 2-panel stacked | (10, 10)–(12, 12) | |
| 3-panel vertical 3×1 | (10, 14)–(12, 16) | Prefer over 1×3 horizontal for maps |
| 4-panel 2×2 | (12, 10)–(14, 12) | |
| 6-panel 2×3 or 3×2 | (16, 10)–(18, 12) | |
| 3+ panels with colorbar | Use `GridSpec` with ratios | See Layout Anti-patterns below |

### Layout Anti-patterns

Common layout mistakes that are **only visible in rendered PNGs** — check every figure for these before entering RR. Any match is an immediate REVISE:

- **Colorbar dominating panels**: In wide 1×N horizontal layouts with a shared colorbar, the colorbar often appears disproportionately large relative to the panels. Fix: use `shrink=0.5–0.7`, or `GridSpec` with explicit `width_ratios` (e.g., `[1, 1, 1, 0.05]` for 3 panels + colorbar). For 3+ map panels, prefer vertical stacking or 2×2 grids over 1×N horizontal.
- **Elements crowding above axes**: Panel labels `(a) subtitle`, quiver keys, and annotations all placed at top-left above axes will overlap. Solution: panel labels at top-left via `set_title(loc="left")`, quiver keys at **top-right** (see [Vectors](#vectors-wind-currents)).
- **Rigid `plt.subplots()` for complex layouts**: Don't force maps + colorbars + legends into a fixed subplot grid. Use `GridSpec` or `subplot_mosaic` with explicit `width_ratios` / `height_ratios` for precise proportional control.
- **Assuming code-level params produce correct output**: figsize, shrink, pad, wspace, hspace are all initial guesses. The only reliable check is the rendered PNG — always verify and adjust in the RR loop.

## Multi-panel Layout

- Simple grids: `fig, axes = plt.subplots(nrows, ncols, subplot_kw={"projection": ...}, constrained_layout=True)`
- Complex layouts (3+ panels, mixed content, shared colorbar needing precise sizing): use `GridSpec` with explicit `width_ratios` / `height_ratios`.
- Shared variable + range → one shared colorbar. Ensure the colorbar is proportionate to panels — control with `shrink`, `aspect`, or a dedicated `GridSpec` column/row.
- Colorbar with units. Font readable at print size, min 7 pt.
- Panel labels: `ax.set_title("(a) Brief subtitle", loc="left", fontsize=11, fontweight="bold")` — lowercase letter in parentheses + short descriptive text (e.g., `"(a) JJA Climatology"`, `"(b) DJF Anomaly"`). NEVER use centered titles or omit the `(a)` prefix.
- **All layout parameters are initial estimates.** Verify in the rendered PNG and adjust — see [Layout Anti-patterns](#layout-anti-patterns) above.

## Maps

- Coastlines by default. National borders if relevant to the study.
- Gridlines: `gl = ax.gridlines(draw_labels=True, linewidth=0.5, alpha=0.5, linestyle="--")` + `gl.top_labels = False; gl.right_labels = False`. Tick font 8–10 pt.

### Borders & Shapefiles

- **National borders**: `~/code/data/map/worldmap/world_boundaries.shp`
- **China map**: `~/code/data/map/chinamap/simple_china.shp`
- **South China Sea inset**: when plotting China maps, add a nine-dash line inset at bottom-right using `~/code/data/map/chinamap/nine_dots.shp`. Use `fig.add_axes([x, y, w, h])` for the inset with a small map extent covering the South China Sea.

### Vectors (wind, currents)

Individual arrows should be distinguishable and reveal spatial structure — no clutter, no empty patches.

- **Skip factors** (by data resolution): 1° → skip 3–5 pts; 0.25° → skip 8–15 pts; 2.5° → skip 1–2 pts. Larger domains need more skipping.
- **Reference magnitude**: round number near the **median** speed (not the max). E.g., typical 5–15 m/s → `10 m/s`.
- **Quiver key**: reference arrow placed **outside the axes at top-right** to avoid conflicting with panel labels `(a)` at top-left.

```python
n = 5  # adjust skip based on resolution and domain size
Q = ax.quiver(lon[::n], lat[::n], u[::n, ::n], v[::n, ::n],
              transform=ccrs.PlateCarree(), scale=200, width=0.003)
# Reference arrow at top-right (top-left reserved for panel labels)
qk = ax.quiverkey(Q, 1.0, 1.04, 10, "10 m/s", labelpos="W",
                  coordinates="axes", fontproperties={"size": 9},
                  labelsep=0.05, zorder=10)
```

### Significance Stippling

The goal is making significant regions identifiable without obscuring the underlying fill.

- **Hatching**: use sparse pattern `"..."` (3 dots); avoid `"/////"` or `"xxxxx"` which are too dense.
- **Scatter**: subsample every 2–4 grid points (`[::3]`), marker size 0.3–1.0, alpha 0.4–0.6.
- For high-resolution data (< 0.5°), increase subsampling to avoid solid-black patches.

```python
# Hatching method
ax.contourf(lon, lat, p_value, levels=[0, 0.05, 1],
            hatches=["...", None], colors="none",
            transform=ccrs.PlateCarree())
# Scatter method (finer control over density)
sig = p_value < 0.05
lon2d, lat2d = np.meshgrid(lon, lat)
ax.scatter(lon2d[sig][::3], lat2d[sig][::3], s=0.5, c="k",
           alpha=0.5, transform=ccrs.PlateCarree(), zorder=5)
```

## Other Figure Types

- **Time series**: legend outside data area, human-readable dates (`DateFormatter`), zero line for anomalies. Pastel palette for multiple lines.
- **Vertical cross-sections**: pressure (hPa) on y-axis with inverted scale (1000 bottom → 100 top). Use `ax.set_yscale("log")` for log-pressure, or linear for troposphere-only. Label isobars clearly.
- **Hovmöller diagrams**: same colormap rules as maps. Time on one axis, space (lat or lon) on the other. Diverging colormap for anomalies.
- **Scatter / regression**: regression line, R² and p-value annotated in the plot. Use `alpha=0.3–0.5` for transparency when point clouds are dense.
- **Taylor diagrams**: correlation on angular axis, std ratio on radial axis. Reference point at (1, 1). Label each model/experiment.
- **Vertical profiles**: pressure on y-axis (inverted), variable on x-axis. Multiple profiles use distinct line styles + legend.

## Export

- `fig.savefig("path.png", dpi=600, bbox_inches="tight")` + `fig.savefig("path.svg", bbox_inches="tight")`.
- Call `plt.close(fig)` after saving to free memory.

## Quick Reject Checklist

**MUST revise immediately** if any is true — common problems that make figures unpublishable:

| Category | Defect |
|---|---|
| **Text & Labels** | Centered title, `suptitle()`, or source/data stamp in figure (these belong in the caption) |
| | Text / ticks / legends / colorbars overlap or clip; any font below 7 pt |
| | Missing axis labels, units, colorbar, or `(a) subtitle` labels on multi-panel |
| **Colormap** | Colormap not from `cmaps` or not using discrete levels |
| | Anomaly/difference not using diverging colormap centered at 0 |
| | Comparable panels with different colormap or level range |
| **Geography** | Wrong projection for region; missing coastlines |
| | White gaps or seams at dateline or plot boundary |
| **Vectors** | Density inappropriate (too dense to distinguish or too sparse to see structure) |
| | Quiver key missing, unlabeled, or poorly positioned |
| **Stippling** | Obscures the main signal or is invisible at print resolution |
| **Contours** | Redundant contour lines on `contourf` without added information |
| **Layout** | Legend outside visible area or occluding data |
| | Pressure y-axis not inverted on cross-sections |
| | Colorbar disproportionately large relative to panels (especially 1×N horizontal layouts) |
| | Panel labels, quiver keys, or annotations overlapping above axes |
| | Layout proportions not verified against rendered PNG (code-level params are unreliable) |
