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

Set scientifically interpretable levels from domain knowledge. Don't auto-scale per panel. If the signal is hidden or saturated, inspect summary ranges, then set common readable levels.

## Projections

| Region | Projection |
|---|---|
| Global | `PlateCarree` or `Robinson` |
| Pacific-centered | `PlateCarree(central_longitude=180)` |
| Tropics | `Mercator` |
| Mid-latitude regional | `PlateCarree` or `LambertConformal` |
| Polar | `NorthPolarStereo` / `SouthPolarStereo` |

## Layout

Starting figure sizes:

| Layout | Size |
|---|---|
| Single map | `(10, 6)`–`(12, 8)` |
| 2-panel side-by-side | `(14, 5)`–`(16, 6)` |
| 2-panel stacked | `(10, 10)`–`(12, 12)` |
| 4-panel | `(12, 10)`–`(14, 12)` |
| 6-panel | `(16, 10)`–`(18, 12)` |

- `constrained_layout=True` for simple grids; `GridSpec` / `subplot_mosaic` for 3+ panels, shared colorbars, or mixed content.
- Panel labels: `ax.set_title("(a) Brief subtitle", loc="left", fontsize=11, fontweight="bold")`. No centered titles, `suptitle()`, source/data stamps.
- Shared variable + range → one shared colorbar; size with `shrink`, `aspect`, or a dedicated narrow `GridSpec` row/column.
- Minimum readable font 7 pt; prefer 8–11 pt for ticks, labels, legends, colorbars.
- For 3+ map panels, prefer vertical stacking or 2×2 / 3×2 over a cramped 1×N row.

## Maps

- Coastlines by default; add national/provincial borders when relevant.
- Gridlines: `draw_labels=True`; disable top/right labels; tick font 8–10 pt.
- Borders: world `~/code/data/map/worldmap/world_boundaries.shp`; China `~/code/data/map/chinamap/simple_china.shp`.
- China maps: add South China Sea / nine-dash inset with `~/code/data/map/chinamap/nine_dots.shp`.

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

## Vectors

Goal: arrows distinguishable, field structure visible.

- Skip factor by resolution: 2.5° → 1–2; 1° → 3–5; 0.25° → 8–15. Larger domains need more skipping.
- Reference magnitude: round value near median speed, not maximum.
- Quiver key outside axes at top-right; top-left is reserved for panel labels.

```python
n = 5
Q = ax.quiver(lon[::n], lat[::n], u[::n, ::n], v[::n, ::n],
              transform=ccrs.PlateCarree(), scale=200, width=0.003)
ax.quiverkey(Q, 1.0, 1.04, 10, "10 m/s", labelpos="W",
             coordinates="axes", fontproperties={"size": 9},
             labelsep=0.05, zorder=10)
```

## Significance

Make significance visible without obscuring the field.

- Hatching: sparse `"..."`; avoid dense `"/////"` or `"xxxxx"`.
- Scatter: subsample every 2–4 grid points; marker size 0.3–1.0; alpha 0.4–0.6.
- High-resolution grids need stronger subsampling to avoid black patches.

```python
ax.contourf(lon, lat, p_value, levels=[0, 0.05, 1],
            hatches=["...", None], colors="none",
            transform=ccrs.PlateCarree())
```

## Other Figure Types

- **Time series** — readable dates; zero line for anomalies; legend outside dense data.
- **Cross-section / profile** — pressure axis inverted (1000 bottom → 100 top); log only when scientifically appropriate.
- **Hovmöller** — same colormap rules as maps; time on one axis, lat/lon on the other.
- **Scatter / regression** — fitted line, R², p-value; alpha for dense clouds.
- **Taylor diagram** — correlation angular axis, std-ratio radial axis, reference at `(1, 1)`.

## Export

```python
fig.savefig("path.png", dpi=600, bbox_inches="tight")
fig.savefig("path.svg", bbox_inches="tight")
plt.close(fig)
```
