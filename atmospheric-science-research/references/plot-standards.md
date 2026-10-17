# Plot Standards

Read before writing or changing plot code. The rendered PNG, not code parameters, is ground truth.

## Size and Fonts

Figure widths are final print widths: 50–85 mm single column, 105–170 mm double column, height under 228 mm. Font sizes are therefore print sizes: panel letters 9–10 pt bold, panel descriptions 8–9 pt, everything else (ticks, labels, legends, colorbar, vector key) 7–9 pt, never below 7 pt.

Arial, with Helvetica as fallback; when patching, keep the project's existing font. Mathtext uses the same family so units stay upright:

```python
plt.rcParams.update({
    "font.family": "sans-serif",
    "font.sans-serif": ["Arial", "Helvetica"],
    "mathtext.default": "regular",
    "mathtext.fontset": "custom",
    "mathtext.rm": "sans",
    "mathtext.it": "sans:italic",
    "mathtext.bf": "sans:bold",
})
```

Units in mathtext superscripts: `r"W m$^{-2}$"`, `r"m s$^{-1}$"`. Never slash notation.

## Colormaps and Levels

`import cmaps`; explicit discrete `levels` for every filled field.

- Absolute fields → sequential, light = low. Temperature: `MPL_YlOrRd`, `cmocean_thermal`. Precipitation / moisture: `CBR_wet`, `MPL_YlGnBu` (white = dry). Other: `viridis`.
- Relative fields (anomaly, difference, trend, regression, correlation) → diverging, white at 0, symmetric `levels` with 0 in the ticks. Temperature and generic: `BlWhRe`, `MPL_RdBu_r`. Precipitation / moisture: `MPL_BrBG`, `CBR_drywet`.
- Never rainbow or hue-wheel maps (`jet`, `rainbow`, `nipy_spectral`, `BlAqGrYeOrRe*`, `temp_19lev`, `wind_17lev`).
- `extend="both"` whenever values can exceed `levels`; `"max"` / `"min"` when one tail is physically closed.
- Comparable panels share `levels`, `cmap`, `extend` (and one `BoundaryNorm` for `pcolormesh`). Never autoscale per panel. Set levels from domain knowledge; if the signal is washed out or saturated, inspect ranges and reset.
- Colorbars: horizontal below maps, vertical for a standalone map or a right-side shared bar; narrow (`aspect` 25–40), never longer than the panel group, few ticks, short label with units. One shared colorbar per variable and scale.

## Layout and Panels

- `constrained_layout=True` for simple grids; `GridSpec` / `subplot_mosaic` for 3+ panels or shared colorbars. Recheck spacing after adding colorbars to GeoAxes.
- Panel letters: lower-case bold `a`, `b`, `c` at each panel's upper-left above the frame: `ax.text(0, 1.02, "a", transform=ax.transAxes, ha="left", va="bottom", fontweight="bold")`.
- Panel descriptions: noun phrases of 6 words or fewer (`850 hPa wind`, `El Niño composite`), centered: `ax.text(0.5, 1.02, ..., ha="center", va="bottom")`. Letters, descriptions, and the vector key share one y per row.
- No `suptitle`, figure caption, default axis title, data stamp, or explanatory text inside the figure; interpretation goes to the caption or README.

## Maps

| Region | Projection |
|---|---|
| Global or Pacific-centered | `PlateCarree(central_longitude=180)` or `Robinson` |
| Tropics and mid-latitude regional | `PlateCarree` (default); `LambertConformal` for mid-latitudes when justified |
| Polar | `NorthPolarStereo` / `SouthPolarStereo` |

- Pass `transform=ccrs.PlateCarree()` to every Cartopy plot call (`contourf`, `contour`, `pcolormesh`, `curly_vector`, `scatter`, `add_geometries`); without it coordinates are treated as projected metres.
- Global fields need a cyclic point:
  ```python
  from cartopy.util import add_cyclic
  cyc, cyc_lon = add_cyclic(da.values, x=da["lon"].values, axis=-1)
  ```
- `ax.coastlines(linewidth=0.8, color="0.2")`; no national borders unless scientifically needed.
- `gl = ax.gridlines(draw_labels=True, linestyle="--", linewidth=0.6, color="0.5", alpha=0.6)`; `gl.top_labels = gl.right_labels = False`. In multi-panel maps label only the left column and bottom row.
- Study-region boxes: dashed red/blue, linewidth 1.2–2.0, high zorder.

## Tibetan Plateau Masking

Applies to plots covering ~25°N–40°N, 70°E–105°E.

Physical mask, every pressure level: points with surface pressure below the level are underground and hold extrapolated values. `ps` and `P_level` in hPa; ERA5 `sp` is in Pa, convert first.

```python
data_masked = data.where(ps >= P_level)   # same for u and v
```

Visual mask, required at 700 hPa and below (700 / 850 / 925 / 1000 hPa): overlay the plateau polygon after filled and vector layers, before boxes and annotations.

```python
from pathlib import Path
import geopandas as gpd

TIBET_SHP = Path(__file__).resolve().parents[1] / "data/map/Tibet/Tibet.shp"
ax.add_geometries(gpd.read_file(TIBET_SHP).geometry, crs=ccrs.PlateCarree(),
                  facecolor="lightgrey", edgecolor="grey", linewidth=0.5, zorder=5)
```

If the file is missing, use the 3000 m orography contour when surface geopotential is available; otherwise mark the figure `BLOCKED`.

## Vectors and Contours

Draw wind and flux vectors with Skyborn's `curly_vector`. It samples the field itself, so do not subsample with `[::n]`.

```python
from skyborn.plot import curly_vector, curly_vector_key

cv = curly_vector(ax, lon, lat, u, v, transform=ccrs.PlateCarree(),
                  density=0.5, ref_magnitude=10, ref_length=0.06,
                  color="#555555", linewidth=0.8, zorder=3)
key = curly_vector_key(ax, cv, U=10, units=r"m s$^{-1}$", labelpos="W",
                       frameon=False, show_description=False,
                       width=0, height=0, padding=0, x=1.0, y=1.02,
                       arrow_props={"color": "#555555", "linewidth": 0.8},
                       text_props={"fontsize": 8})
fig.canvas.draw()                    # after the final layout change
key.x = 1.0 - key.patch.get_width()  # key arrow tip on the map's right edge
```

- `density`: start at 0.5; for non-PlateCarree projections add `regrid_shape=20–30`.
- Always pass `ref_magnitude` (one value near the median speed, also the key's `U`); without it each panel scales to its own maximum. `ref_length` is a fraction of axes width.
- Always pass `units` in mathtext; the default `"m/s"` breaks the unit rule.
- One key per figure, top-right outside the map on the letter/description baseline, `arrow_props` matching the field arrows. Panels sharing a key use the same `ref_magnitude` and `ref_length`; if their widths differ, pass `ref_length_px` instead (if the installed Skyborn rejects it, give each panel its own key).
- Contours only when they add interpretation (zero lines, thresholds). Zero contour dark, 0.8–1.0 pt; others 0.4–0.6 pt with moderate alpha; label selected contours only.

## Significance

Visible but subordinate to the field. Default to subsampled dots:

```python
step = 3  # every 2–4 grid points
lon2, lat2 = np.meshgrid(lon[::step], lat[::step])
sig = p_value[::step, ::step] < 0.05
ax.scatter(lon2[sig], lat2[sig], s=0.6, c="k", alpha=0.5, linewidths=0,
           transform=ccrs.PlateCarree())
```

Tune size and alpha against the PNG so dots never merge into dark patches. Use low-saturation red/blue only when the sign of the response must be distinguished. Hatching alternative: sparse `"..."` via `contourf(..., levels=[0, 0.05, 1], hatches=["...", None], colors="none")`; never dense hatching.

## Other Figure Types

- Time series: thin lines, zero or reference line for anomalies, uncertainty as shaded bands.
- Cross-section / profile: pressure axis inverted (1000 hPa at the bottom); topography filled grey.
- Correlation matrices: diverging, centered at 0, limits `[-1, 1]`; significance as dots or outlines, not numbers in every cell.
- EOF: diverging maps centered at 0; PCs normalized with a zero line; explained variance in the panel description; state the sign convention.
- Scatter / regression: annotate r, slope, or p only when central to the result.

## Export

Each plot script takes an optional `--final` flag: without it a 300 dpi PNG for RR, with it a 600 dpi PNG and a PDF.

```python
exts = ("png", "pdf") if final else ("png",)
for ext in exts:
    fig.savefig(f"{stem}.{ext}", dpi=600 if final else 300,
                bbox_inches="tight", pad_inches=0.05)
plt.close(fig)
```

If the PDF is large or slow, pass `rasterized=True` to dense layers.
