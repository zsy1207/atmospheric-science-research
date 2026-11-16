---
name: atmospheric-science-plotting
description: Publication-grade plotting for atmospheric and climate science.
---

# Atmospheric Science Plotting

Plot local atmospheric and climate data to journal standard, inside a reproducible project.

## Workflow

New analyses run steps 1–7. Figure-only fixes use Patch Mode.

1. **Probe** — Inspect metadata only: `ncdump -h`, `cdo sinfon`, `h5dump -H`, or a metadata-only Python one-liner for Zarr.
2. **Plan** — Ask only scientific questions that change the result, each with a recommended answer. Record every adopted assumption in the plan and the README.
3. **Build** — `figN/compute_*.py`, `figN/plot_*.py`, `data_output/figN/...`. Do not reorganize existing projects unless asked.
4. **Compute** — Scripts run without required CLI args, are idempotent, and write reusable outputs before plots read them. Print each output's min, max, NaN fraction, and units, so a wrong variable, unit, or sign is caught before plotting.
5. **Plot** — Follow Plot Standards.
6. **Review** — Run RR until each figure is `PASS` or `BLOCKED`.
7. **Export and document** — Export final files (Plot Standards › Export), then write or update one Chinese `README.md` from the README Template. `BLOCKED` figures and unresolved risks go in its risk section.

## Compute Rules

- **Speed and efficiency** — Prefer packages and algorithms that improve computational efficiency, such as dask.
- **Regridding** — `cdo remapcon` / xesmf `"conservative"` for precipitation and fluxes; `remapbil` / `"bilinear"` for smooth state variables (T, Z, u, v, q).

## Plot Standards

The rendered PNG, not code parameters, is ground truth. Set every parameter below to fit the actual project and what RR finds; the goal is a high-quality scientific figure ready for publication in a top-tier atmospheric-science journal.

### Size and Fonts

Figure widths are final print widths: 50–85 mm single column, 105–170 mm double column, height under 228 mm; `figsize` takes inches, so pass `(width_mm / 25.4, height_mm / 25.4)`. Font sizes are therefore print sizes: panel letters 12 pt bold, panel descriptions 10 pt, everything else (ticks, labels, legends, colorbar, vector key) 8–9 pt, never below 8 pt.

Arial, with Helvetica as fallback; when patching, keep the project's existing font. Mathtext uses the same family so units stay upright, and the PDF embeds TrueType fonts so its text stays editable:

```python
plt.rcParams.update({
    "font.family": "sans-serif",
    "font.sans-serif": ["Arial", "Helvetica"],
    "font.size": 8,
    "mathtext.default": "regular",
    "mathtext.fontset": "custom",
    "mathtext.rm": "sans",
    "mathtext.it": "sans:italic",
    "mathtext.bf": "sans:bold",
    "axes.linewidth": 0.6,
    "xtick.major.width": 0.6,
    "ytick.major.width": 0.6,
    "legend.frameon": False,
    "pdf.fonttype": 42,
})
```

Units in mathtext superscripts: `r"W m$^{-2}$"`, `r"m s$^{-1}$"`. Never slash notation.

### Colormaps and Levels

Use explicit discrete `levels` for every filled field: roughly 8–14 bins with round boundaries. Signed fields where 0 is meaningful take a diverging map, one-signed fields a sequential map. Colormaps come only from this table (`_r` reverses):

| Field | Colormap |
|---|---|
| Temperature, SST, height, SLP, heat flux, radiation: anomaly, difference, trend, regression, correlation | `RdBu_r` (warm / high is red); or `cmaps.BlueWhiteOrangeRed`, `cmaps.hotcold_18lev`, `bwr` |
| Precipitation, humidity, soil moisture, cloud: the same signed quantities | `BrBG` (wet is green); `BrBG_r` when positive means drying, e.g. moisture-flux divergence |
| Raw u, v, ω, vorticity, divergence, streamfunction, net radiation | `RdBu_r`; `PuOr_r` or `PRGn` when the figure already uses `RdBu_r` for another variable |
| Raw temperature, heatwave metrics | `YlOrRd` |
| Raw precipitation | `cmaps.precip3_16lev` or `YlGnBu` |
| Raw humidity, precipitable water, cloud | `YlGnBu` |
| Wind speed, variance, other magnitudes | `viridis` or `plasma` |
| Raw height, SLP | line contours over the filled field, not a fill |

- Diverging maps: symmetric `levels` with no level at 0, so the middle bin straddles 0 and is white (e.g. ±0.5, ±1, …); keep the colorbar's default equal-width bins and put ticks on the level boundaries, so 0 itself is not labeled.
- `extend="both"` whenever values can exceed `levels`; `"max"` / `"min"` when one tail is physically closed.
- Comparable panels share `levels`, `cmap`, `extend` (for `pcolormesh`, one `BoundaryNorm(levels, cmap.N, extend="both")` with `cmap = plt.get_cmap(name)`). Never autoscale per panel. Set levels from domain knowledge; if the signal is washed out or saturated, inspect ranges and reset.
- Colorbars: horizontal below maps, vertical for a standalone map or a right-side shared bar; narrow (`aspect` 20–40 by bar length), close to the panels (`pad` about 0.02 of the panel group's width or height; the default is loose), never longer than the panel group, few ticks, short label with units. One shared colorbar per variable and scale.

### Layout and Panels

- Always create figures with `dpi=600`, so the vector key is measured at the export resolution, and `layout="compressed"` for map grids or `layout="constrained"` otherwise; use `GridSpec` / `subplot_mosaic` inside it for 3+ panels or shared colorbars.
- Maps have a fixed aspect: a figure too tall leaves white bands between rows, one too short leaves them between columns. Start tall under `"compressed"`, set the gaps, and trim the height to the content:
  ```python
  fig.get_layout_engine().set(w_pad=0.08, h_pad=0.08)  # inches around each panel: about 4 mm between panels
  fig.canvas.draw()
  fig.set_size_inches(fig.get_figwidth(), fig.get_tightbbox().height + 0.1)
  ```
- Panel letters: lower-case bold `a`, `b`, `c` at each panel's upper-left above the frame: `ax.text(0, 1.02, "a", transform=ax.transAxes, ha="left", va="bottom", fontsize=12, fontweight="bold")`.
- Panel descriptions: noun phrases of 6 words or fewer (`850 hPa wind`, `El Niño composite`), centered: `ax.text(0.5, 1.02, "850 hPa wind", transform=ax.transAxes, ha="center", va="bottom", fontsize=10)`. Letters, descriptions, and the vector key share one y per row. In a grid of variables × cases, write each case once above its column and each variable once left of its row (`ax.set_ylabel`) instead of repeating them.
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
- Regions crossing 0° or 180°: before subsetting, convert longitudes to −180–180 (`(da.lon + 180) % 360 - 180`) or 0–360 (`da.lon % 360`) with `assign_coords`, then `sortby("lon")`.
- `ax.coastlines("110m", linewidth=0.5, color="0.2")`; `"50m"` only for panels spanning under about 60° of longitude, since its small islands speckle wider maps. No national borders unless scientifically needed.
- Missing data shows grey, never white, which already means near zero: `ax.set_facecolor("0.85")`.
- Rectangular projections take frame ticks at round values inside the frame and no interior gridlines; multi-panel maps label only the left column and bottom row:
  ```python
  from cartopy.mpl.ticker import LatitudeFormatter, LongitudeFormatter
  ax.set_xticks(lon_ticks, crs=ccrs.PlateCarree())
  ax.set_yticks(lat_ticks, crs=ccrs.PlateCarree())
  ax.xaxis.set_major_formatter(LongitudeFormatter())
  ax.yaxis.set_major_formatter(LatitudeFormatter())
  ax.tick_params(labelleft=col == 0, labelbottom=row == nrows - 1)
  ```
- Other projections keep thin gridlines: `gl = ax.gridlines(draw_labels=True, linestyle="--", linewidth=0.4, color="0.5", alpha=0.6)`; `gl.top_labels = gl.right_labels = False`.
- Study-region boxes: dashed red/blue, linewidth 1.2–2.0, high zorder.

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
- Mask weak vectors before plotting, since they render as stubs that clutter the map: below about 20% of `ref_magnitude`, and in composites and regressions also those failing the test. `speed = np.hypot(u, v); u, v = u.where(speed >= 2), v.where(speed >= 2)`.
- Pass `units` in mathtext; the default `"m/s"` breaks the unit rule.
- One key at the top-right outside the map, on the letter/description baseline, styled like the field arrows.
- Contours only when they add interpretation (zero lines, thresholds). Zero contour dark, 0.8–1.0 pt; others 0.4–0.6 pt with moderate alpha; label selected contours only.

### Significance

Visible but subordinate to the field. Draw dots with `scatter` on a thinned grid, so spacing, size, and grey level stay tunable during RR:

```python
sig = (p < 0.05)[::n, ::n]
lon2d, lat2d = np.meshgrid(sig["lon"], sig["lat"])
ax.scatter(lon2d[sig.values], lat2d[sig.values], s=3, c="0.2", linewidths=0,
           transform=ccrs.PlateCarree())
```

Dots should never merge into dark patches or hide the field. Use low-saturation red/blue only when the sign of the response must be distinguished.

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

Never pass `bbox_inches="tight"`: it crops the canvas, so the width and font sizes no longer match print. Rasterize filled fields (`filled.set_rasterized(True)`) and keep text, coastlines, and vectors as vector: PDF viewers otherwise show white hairlines between `contourf` polygons.

## RR Rules

Review-revision (RR) catches visual defects and obvious data errors in each rendered figure.

- Open the rendered PNG every iteration; reviewing code alone never counts.
- Check it against Plot Standards and the checklist below, then classify: `PASS` (nothing left), `REVISE` (fix every visible issue that needs no new information, re-render once, repeat), or `BLOCKED` (needs a user decision or missing input, or the output is unreadable).
- At least one iteration before `PASS`, at most 5. If not `PASS` by then, mark `BLOCKED` and list what remains in the README.
- Numbers in Plot Standards are starting values: adjust line widths, densities, dot size and spacing, levels, and figure height to what this figure needs.
- Revise anything that would look out of place in a top-tier atmospheric-science journal, even when no rule fails.
- Where subagents are available, let a fresh one that sees only the PNG and this section judge `PASS`; the author is a lenient reviewer.

| Check | Pass when |
|---|---|
| Text | Arial or the project font; mathtext units; nothing clipped or below 8 pt; no suptitle, caption, or explanatory text |
| Panels | Bold letters, short centered descriptions, and the vector key on one baseline |
| Colormap | The variable's row in the table, with explicit `levels`; white bin straddles 0 on diverging maps; `extend` set; signal neither washed out nor saturated; comparable panels share one scale |
| Maps | Coastlines; lat/lon labels on the left column and bottom row only; missing data grey; no dateline seam |
| Vectors | Readable density; no weak-wind stubs; one key at top-right with its tip on the right edge |
| Significance | Visible but subordinate to the field |
| Layout | Panels fill the canvas with even gaps of a few mm; colorbar easy to read and close to the panels; no overlap |
| Physics | Plausible values, units, and signs, consistent with the ranges printed at compute time; pressure axis inverted |
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
