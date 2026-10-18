---
name: atmospheric-science-research
description: Computation and publication-grade plotting for atmospheric and climate research on local data files (NetCDF, GRIB, HDF5, Zarr; ERA5, CMIP, reanalysis). Use whenever the user works with gridded lat/lon or pressure-level data and needs diagnostics (climatology, anomalies, composites, regression/correlation, trends, EOF, extremes) or figures (Cartopy maps, cross-sections, time series), even if they do not say "paper figure".
---

# Atmospheric Science Research

Compute diagnostics on local atmospheric data and plot them to journal standard, inside a reproducible project.

## Workflow

New analyses run steps 1–7. Figure-only fixes use Patch Mode.

1. **Probe** — Inspect metadata only: `ncdump -h`, `cdo sinfon`, `h5dump -H`, or a metadata-only Python one-liner for Zarr.
2. **Plan** — Ask only scientific questions that change the result, each with a recommended answer. Record every adopted assumption in the plan and the README.
3. **Build** — `figN/compute_*.py`, `figN/plot_*.py`, `data_output/figN/...`. Do not reorganize existing projects unless asked.
4. **Compute** — Scripts run without required CLI args, are idempotent, and write reusable outputs before plots read them.
5. **Plot** — Follow Plot Standards.
6. **Review** — Run RR until each figure is `PASS` or `BLOCKED`.
7. **Export and document** — Export final files (Plot Standards › Export), then write or update one Chinese `README.md` from the README Template. `BLOCKED` figures and unresolved risks go in its risk section.

## Compute Rules

- **Speed and efficiency** — Prefer packages and algorithms that improve computational efficiency, such as dask.
- **Regridding** — `cdo remapcon` / xesmf `"conservative"` for precipitation and fluxes; `remapbil` / `"bilinear"` for smooth state variables (T, Z, u, v, q).
- **Regional padding** — Pad subsets by at least 1 grid point beyond the target bounds so maps have no blank border.

## Plot Standards

The rendered PNG, not code parameters, is ground truth. Set every parameter below to fit the actual project and what RR finds; the goal is a high-quality scientific figure ready for publication in a top-tier atmospheric-science journal.

### Size and Fonts

Figure widths are final print widths: 50–85 mm single column, 105–170 mm double column, height under 228 mm. Font sizes are therefore print sizes: panel letters 12 pt bold, panel descriptions 10 pt, everything else (ticks, labels, legends, colorbar, vector key) 8–9 pt, never below 8 pt.

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

### Colormaps and Levels

Use explicit discrete `levels` for every filled field. Colormaps come only from this whitelist (reversed `_r` versions allowed, e.g. `RdBu_r` so negatives are blue). Within the whitelist, pick the colormap whose colors match the variable's convention:

- Signed fields where 0 is meaningful (anomalies, differences, trends, regression, correlation, and raw ω, vorticity, divergence, u, v, moisture-flux divergence, net radiation) → diverging: `RdBu`, `BrBG`, `bwr`, `PuOr`, `PRGn`, `cmaps.BlueWhiteOrangeRed`, `cmaps.hotcold_18lev`. Use symmetric `levels` with no level at 0, so the middle bin straddles 0 and is white (e.g. ±0.5, ±1, …); draw the colorbar with `spacing="proportional"` and tick 0 at the middle of the white bin.
- Non-anomaly heatwave, precipitation, humidity, and wind-speed fields → sequential: `plasma`, `YlGnBu`, `viridis`, `YlOrRd`.
- `extend="both"` whenever values can exceed `levels`; `"max"` / `"min"` when one tail is physically closed.
- Comparable panels share `levels`, `cmap`, `extend` (for `pcolormesh`, one `BoundaryNorm(levels, cmap.N, extend="both")` with `cmap = plt.get_cmap(name)`). Never autoscale per panel. Set levels from domain knowledge; if the signal is washed out or saturated, inspect ranges and reset.
- Colorbars: horizontal below maps, vertical for a standalone map or a right-side shared bar; narrow (`aspect` 25–40), never longer than the panel group, few ticks, short label with units. One shared colorbar per variable and scale.

### Layout and Panels

- Always create figures with `dpi=600, layout="constrained"`, so the vector key is measured at the export resolution; use `GridSpec` / `subplot_mosaic` inside it for 3+ panels or shared colorbars. Recheck spacing after adding colorbars to GeoAxes.
- Panel letters: lower-case bold `a`, `b`, `c` at each panel's upper-left above the frame: `ax.text(0, 1.02, "a", transform=ax.transAxes, ha="left", va="bottom", fontweight="bold")`.
- Panel descriptions: noun phrases of 6 words or fewer (`850 hPa wind`, `El Niño composite`), centered: `ax.text(0.5, 1.02, "850 hPa wind", transform=ax.transAxes, ha="center", va="bottom")`. Letters, descriptions, and the vector key share one y per row.
- No `suptitle`, figure caption, default axis title, data stamp, or explanatory text inside the figure; interpretation goes to the caption or README.

### Maps

| Region | Projection |
|---|---|
| Global or Pacific-centered | `PlateCarree(central_longitude=180)` or `Robinson` |
| Tropics and mid-latitude regional | `PlateCarree` (default); `LambertConformal` for mid-latitudes when justified |
| Polar | `NorthPolarStereo` / `SouthPolarStereo` |

- Data calls (`contourf`, `contour`, `pcolormesh`, `curly_vector`, `scatter`) take `transform=ccrs.PlateCarree()`; without it coordinates are read in the projection's own units and the field is silently misplaced. `add_geometries` takes `crs=ccrs.PlateCarree()`; panel text takes `transform=ax.transAxes`.
- Global fields need a cyclic point:
  ```python
  from cartopy.util import add_cyclic
  cyc, cyc_lon = add_cyclic(da.values, x=da["lon"].values, axis=-1)
  ```
- Regions crossing 0° or 180°: before subsetting, convert longitudes to −180–180 (`(da.lon + 180) % 360 - 180`) or 0–360 (`da.lon % 360`) with `assign_coords`, then `sortby("lon")`; padding alone cannot close the seam.
- `ax.coastlines(linewidth=0.8, color="0.2")`; no national borders unless scientifically needed.
- `gl = ax.gridlines(draw_labels=True, linestyle="--", linewidth=0.6, color="0.5", alpha=0.6)`; `gl.top_labels = gl.right_labels = False`. In multi-panel maps label only the left column and bottom row: `gl.left_labels = col == 0; gl.bottom_labels = row == nrows - 1`.
- Study-region boxes: dashed red/blue, linewidth 1.2–2.0, high zorder.

### Tibetan Plateau Masking

On maps that include the Tibetan Plateau, at 700 hPa and below (700 / 850 / 925 / 1000 hPa): mask vectors, contour lines, and significance dots on the plateau with `regionmask`, keep the filled field complete, and draw the grey polygon over it. The polygon's default zorder (1.5) already sits above the fill, so no white edge appears.

```python
from pathlib import Path
import geopandas as gpd, regionmask

tibet = gpd.read_file(Path(__file__).resolve().parents[1] / "data/map/Tibet/Tibet.shp")
on_tp = regionmask.mask_geopandas(tibet, u).notnull()   # True on plateau grid points
u, v = u.where(~on_tp), v.where(~on_tp)                 # same for contour fields and p-values
ax.add_geometries(tibet.geometry, crs=ccrs.PlateCarree(),
                  facecolor="lightgrey", edgecolor="grey", linewidth=0.5)
```

If the file is missing, use the 3000 m orography contour when surface geopotential is available; otherwise mark the figure `BLOCKED`.

### Vectors and Contours

Draw wind and flux vectors with Skyborn's `curly_vector`; it samples the field itself, so do not subsample with `[::n]`. The values below are starting points: tune density, arrow length, line width, and the key during RR until the flow reads clearly at print size.

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

- Pass `ref_magnitude` (an interpretable speed, also the key's `U`); without it each panel scales to its own maximum. Panels sharing one key must have equal widths and the same `ref_magnitude` and `ref_length`; otherwise give each panel its own key. Build the key from a panel whose maximum speed is at least `U`, since Skyborn caps arrow length at that maximum. For non-PlateCarree projections, `regrid_shape` evens out arrow spacing.
- Pass `units` in mathtext; the default `"m/s"` breaks the unit rule.
- One key at the top-right outside the map, on the letter/description baseline, styled like the field arrows.
- Contours only when they add interpretation (zero lines, thresholds). Zero contour dark, 0.8–1.0 pt; others 0.4–0.6 pt with moderate alpha; label selected contours only.

### Significance

Visible but subordinate to the field. Use dots by default; choose their spacing, size, and alpha for the panel and tune them during RR. Dots should never merge into dark patches or hide the field. Use low-saturation red/blue only when the sign of the response must be distinguished. Sparse `"."` or `".."` hatching via `contourf(..., levels=[0, 0.05, 1], hatches=[".", None], colors="none")` is an alternative.

### Other Figure Types

- Time series: thin lines, zero or reference line for anomalies, uncertainty as shaded bands.
- Cross-section / profile: pressure axis inverted (1000 hPa at the bottom); topography filled grey.
- Correlation matrices: diverging, centered at 0, limits `[-1, 1]`; significance as dots or outlines, not numbers in every cell.
- EOF: diverging maps centered at 0; PCs normalized with a zero line; explained variance in the panel description; state the sign convention.
- Scatter / regression: annotate r, slope, or p only when central to the result.

### Export

Everything renders at 600 dpi. Each plot script takes an optional `--final` flag: without it a PNG for RR, with it a PNG and a PDF.

```python
exts = ("png", "pdf") if final else ("png",)
for ext in exts:
    fig.savefig(f"{stem}.{ext}", dpi=600)
plt.close(fig)
```

If the PDF is large or slow, pass `rasterized=True` to dense layers.

## RR Rules

Review-revision (RR) catches visual defects and obvious data errors in each rendered figure.

- Open the rendered PNG every iteration; reviewing code alone never counts.
- Check it against Plot Standards and the checklist below, then classify: `PASS` (nothing left), `REVISE` (fix every visible issue that needs no new information, re-render once, repeat), or `BLOCKED` (needs a user decision or missing input, or the output is unreadable).
- At least one iteration before `PASS`, at most 5. If not `PASS` by then, mark `BLOCKED` and list what remains in the README.
- Tune vector density and key, and stippling spacing and size, here against the PNG; the plot code only sets starting values.
- Revise anything that would look out of place in a top-tier atmospheric-science journal, even when no rule fails.

| Check | Pass when |
|---|---|
| Text | Arial or the project font; mathtext units; nothing clipped or below 8 pt; no suptitle, caption, or explanatory text |
| Panels | Bold letters, short centered descriptions, and the vector key on one baseline |
| Colormap | Whitelisted colormap with explicit `levels`; 0-centered diverging for signed fields with a meaningful zero, sequential otherwise; `extend` set; comparable panels share one scale |
| Maps | Coastlines; lat/lon labels; no dateline seam or blank border |
| Tibet | At 700 hPa and below, no arrows, contours, or dots on the plateau; grey polygon drawn |
| Vectors | Readable density; one key at top-right with its tip on the right edge |
| Significance | Visible but subordinate to the field |
| Layout | Balanced panels; narrow colorbar; no overlap |
| Physics | Plausible values, units, and signs; pressure axis inverted |
| Overall | A high-quality scientific figure fit for a top-tier atmospheric-science journal |

## README Template

Write one Chinese `README.md` once every figure is `PASS` or `BLOCKED`. Match the actual project layout; do not restructure the project to fit the template.

```markdown
# 项目说明

简要说明科学目标和主要工作内容。

## 1. 数据简述

- 输入数据路径：
- 数据来源与类型：
- 时间范围：
- 空间范围与分辨率：

## 2. 路径结构和实现功能

- `data_output/fig1/xxx.nc`：实现功能
- `fig1/compute_xxx.py`：实现功能
- `fig1/plot_xxx.py`：实现功能
- `fig1/fig1_xxx.png`、`fig1/fig1_xxx.pdf`：实现功能

## 3. 实现算法与细节

- 采用的假设与默认选择：逐条列出用户确认的选择和可逆的默认设置，算法简单时也要写。
- 算法如无需展开，可写：`本任务算法直接，省略。`
- 若有复杂处理，写清关键公式、统计方法、重网格方案和加速策略。

## 4. 可能存在的问题与风险

- 列出 `BLOCKED` 的图、原因和剩余问题，以及其他未解决的风险。
- 无则写：`目前无明显问题。`

## 5. 图片简述

- 以大气科学论文写作口径，专业、简洁地说明每张图的内容与意义。

## 6. 版本更新

- （更新时填写日期、变更内容、影响文件）
```

On later modification, including Patch Mode, append to「版本更新」and update only the affected path and figure sections.

## Patch Mode

Read the existing code, make the smallest safe change, re-render only the affected figures, run RR, export, and append a「版本更新」entry to the README.

If RR reveals a compute bug (wrong variable, unit error, bad climatology baseline), stop patching and restart from step 2.
