# Plot Standards

## Colormap table

ALL colormaps MUST come from `import cmaps` with discrete `levels`. NEVER continuous interpolation. NEVER `MPL_jet`, `MPL_rainbow`, `MPL_viridis`, `NCV_jet`.

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

Comparable panels MUST share the same colormap and range unless units differ.

## Projections

| Region | Projection |
|---|---|
| Global | `PlateCarree` or `Robinson` |
| Tropics | `Mercator` |
| Mid-latitude regional | `PlateCarree` or `LambertConformal` (set `central_longitude`, `standard_parallels`) |
| Polar | `NorthPolarStereo` / `SouthPolarStereo` |
| Pacific-centered | `PlateCarree(central_longitude=180)` |

## Figure sizing

| Layout | `figsize` (inches) |
|---|---|
| Single map | (10, 6)–(12, 8) |
| Single time series | (10, 4)–(12, 5) |
| 2-panel side-by-side | (14, 5)–(16, 6) |
| 2-panel stacked | (10, 10)–(12, 12) |
| 3-panel horizontal 1×3 | (16, 5)–(18, 6) |
| 3-panel vertical 3×1 | (10, 14)–(12, 16) |
| 4-panel 2×2 | (12, 10)–(14, 12) |
| 6-panel | (16, 10)–(18, 12) |
| Map + time series | (10, 8)–(12, 10) |

## Multi-panel

- `fig, axes = plt.subplots(nrows, ncols, subplot_kw={"projection": ...}, constrained_layout=True)`
- Shared variable + range → one shared colorbar at bottom or right.
- Every colorbar shows units. Font: DejaVu Sans, 10–12 pt (single) / 8–10 pt (multi), min 7 pt.
- Panel labels: `ax.text(-0.02, 1.03, "(a)", transform=ax.transAxes, fontsize=11, fontweight="bold", va="bottom", ha="right")`

## Maps

- Coastlines by default. National borders ONLY if requested.
- Gridlines: `gl = ax.gridlines(draw_labels=True, linewidth=0.5, alpha=0.5, linestyle="--")` + `gl.top_labels = False; gl.right_labels = False`. Tick font 8–10 pt.
- Vectors: density must make individual arrows distinguishable yet reveal spatial structure — no visual clutter, no empty patches.
  - 1° data → skip 3–5 pts; 0.25° data → skip 8–15 pts; 2.5° data → skip 1–2 pts. Adjust by domain size: larger domain needs more skipping.
  - Quiver key (reference arrow + label): place inside a **white opaque box flush against the lower-right border** of the axes. Arrow size must be proportionate — fits inside the box without overflow.
  - Reference magnitude: choose a round number near the **median** wind speed in the plot (not the max). E.g., if typical values are 5–15 m/s, use `10 m/s`; if 0.5–3 m/s, use `2 m/s`.

```python
n = 5  # adjust skip based on resolution and domain
Q = ax.quiver(lon[::n], lat[::n], u[::n, ::n], v[::n, ::n],
              transform=ccrs.PlateCarree(), scale=200, width=0.003)
# White box flush against lower-right corner
import matplotlib.patches as mpatches
rect = mpatches.FancyBboxPatch(
    (0.84, 0.0), 0.16, 0.1, transform=ax.transAxes,
    facecolor="white", edgecolor="k", linewidth=0.5,
    zorder=9, boxstyle="square,pad=0.01")
ax.add_patch(rect)
qk = ax.quiverkey(Q, 0.92, 0.04, 10, "10 m/s", labelpos="S",
                  coordinates="axes", fontproperties={"size": 9},
                  labelsep=0.05, zorder=10)
```

- Significance stippling — density must make significant regions identifiable without obscuring the fill beneath:
  - Hatching: use sparse pattern `"..."` (3 dots); avoid `"/////"` or `"xxxxx"` which are too dense.
  - Scatter: subsample every 2–4 grid points (`[::3]`), marker size 0.3–1.0, alpha 0.4–0.6.
  - For high-resolution data (< 0.5°), increase subsampling to avoid solid-black patches.

```python
# Hatching method
ax.contourf(lon, lat, p_value, levels=[0, 0.05, 1],
            hatches=["...", None], colors="none",
            transform=ccrs.PlateCarree())
# Scatter method (finer control)
sig = p_value < 0.05
lon2d, lat2d = np.meshgrid(lon, lat)
ax.scatter(lon2d[sig][::3], lat2d[sig][::3], s=0.5, c="k",
           alpha=0.5, transform=ccrs.PlateCarree(), zorder=5)
```

## Other figure types

- **Time series**: legend, human-readable dates, zero line for anomalies. Pastel palette.
- **Vertical cross-sections**: pressure (hPa) y-axis MUST be inverted (1000 bottom → 100 top).
- **Hovmöller**: same colormap rules. Time on one axis, space on the other.
- **Scatter / regression**: regression line, R², p-value annotated. Transparency for dense clouds.

## Export

- `fig.savefig("path.png", dpi=600, bbox_inches="tight")` + `fig.savefig("path.svg", bbox_inches="tight")`.
- Call `plt.close(fig)` after saving to free memory.

## Quick reject checklist

Revise IMMEDIATELY if any is true:
- Main title or source/data stamp in figure
- Text / ticks / legends / colorbars overlap or clip; font below 7 pt
- Missing axis labels, units, colorbar, or `(a)`/`(b)` labels on multi-panel
- Colormap not from `cmaps` or not using discrete levels
- Anomaly/difference not using diverging cmap centered at 0
- Comparable panels with different color scales
- Wrong projection for region; missing coastlines
- Vector density inappropriate (too dense to distinguish or too sparse to see structure); quiver key missing, unlabeled, or not inside lower-right of axes
- Stippling obscures the main signal or is invisible at print resolution
- Redundant contour lines on `contourf` without added information
- White gaps or seams at dateline or plot boundary
- Legend outside visible area or occluding data
- Pressure y-axis not inverted on cross-sections
