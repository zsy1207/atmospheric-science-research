# Plot Standards

Use this file for any figure rendering or any `RR` step.

## 1. Core defaults

- Aim for publication-quality figures at the highest standard in atmospheric science.
- Keep in-figure text to the minimum needed to read the science.
- The image created should be professional, scientific, and informative, while maintaining an attractive visual style.
- Do not use a main title.
- Use subfigure labels (a), (b), (c), with brief subtitles placed just outside the upper-left corner of each subfigure.
- Do not allow text, ticks, legends, colorbars, or panel labels to overlap or clip.
- Use a white background and restrained styling. Do not use 3D effects, shadows, decorative gradients, or patterned fills.
- Use 'Arial' as the default font.
- Use English in-figure text by default for journal-style outputs unless the user or project requires otherwise.
- Do not place a large title or a source/process stamp inside the figure. Labels such as `ERA5 monthly mean` belong in the caption or `README.md`, not in the graph.
- Text, axis labels, legends, and colorbars must remain readable at final output size. For single-panel figures, use 10–12 pt; for multi-panel figures, use 8–10 pt; never go below 7 pt.
- Keep the image clean and clear. Do not add any meaningless labels or text. If a variable is shown with a filled contour plot, do not add contour lines on top unless they convey additional information (e.g., geopotential height contours over a temperature fill).

## 2. Colormap selection guide

When a colormap is used, it must come from the `cmaps` package and must be applied with discrete coordinates or levels.

### 2.1 Variable-to-colormap mapping

| Variable / context | Style | Recommended `cmaps` choices | Notes |
|---|---|---|---|
| Temperature (absolute) | Sequential warm | `temp_19lev`, `hotcolr_19lev` | Cool-to-warm progression |
| Temperature (anomaly / difference) | Diverging, white at 0 | `temp_diff_18lev`, `BlueWhiteOrangeRed`, `BlWhRe` | Symmetric range for centered anomalies |
| Precipitation (absolute) | Sequential wet | `precip_11lev`, `precip3_16lev`, `precip2_17lev`, `CBR_wet` | White/light for dry end |
| Precipitation (anomaly / difference) | Diverging brown–white–green | `precip_diff_12lev`, `CBR_drywet`, `GMT_drywet` | Brown = dry, green = wet |
| Wind speed | Sequential | `wind_17lev` | Light-to-dark |
| SST (absolute) | Sequential warm | `temp_19lev`, `cmocean_thermal` | Same as temperature |
| SST (anomaly) | Diverging | `GHRSST_anomaly`, `MPL_sstanom`, `temp_diff_18lev` | Center at 0 |
| Geopotential height | Sequential or multi-hue | `BlAqGrYeOrRe`, `BlAqGrYeOrReVi200` | Match synoptic conventions |
| Humidity / moisture | Sequential blue-green | `MPL_BuGn`, `MPL_YlGnBu`, `cmocean_haline` | |
| OLR / radiation | Sequential warm | `sunshine_9lev`, `MPL_YlOrRd` | Light = low, dark = high |
| Radiation (anomaly) | Diverging | `sunshine_diff_12lev`, `BlueWhiteOrangeRed` | Center at 0 |
| Vorticity / divergence | Diverging | `BlWhRe`, `NCV_blu_red`, `hotcold_18lev` | Center at 0 |
| Correlation / regression coeff. | Diverging | `NCV_blu_red`, `BlueWhiteOrangeRed`, `MPL_RdBu_r` | Center at 0, symmetric ±1 or ±max |
| Pressure / MSLP | Sequential | `BlAqGrYeOrRe` | Low-to-high |
| Topography | Terrain | `GMT_topo`, `topo_15lev`, `GMT_relief` | Follow topographic convention |
| Vegetation / NDVI | Green sequential | `NEO_modis_ndvi`, `vegetation_modis` | |
| Generic anomaly / bias / difference | Diverging, white at 0 | `BlueWhiteOrangeRed`, `BlWhRe`, `NCV_blu_red` | Always center at 0 |
| Generic absolute field | Sequential | `BlAqGrYeOrReVi200`, `MPL_YlOrRd` | Choose based on physical context |

### 2.2 Selection principles

- Diverging colormaps: center white at 0 for anomaly, difference, bias, correlation, or any field where sign matters. Use symmetric ranges when the field is centered.
- Sequential colormaps: use for absolute quantities that increase in one direction.
- Avoid perceptually non-uniform colormaps (`MPL_jet`, `MPL_rainbow`, `MPL_viridis`, `NCV_jet`) unless field convention demands it.
- When comparing panels, use the same colormap and range.
- If compared quantities have different units or intrinsic scales, separate colorbars are allowed.
- Use discrete levels, not continuous interpolation. Choose range and interval scientifically for the variable and comparison target.

## 3. Maps

### 3.1 Geospatial setup

- Draw `coastlines` by default. Do not draw national borders unless the user asks or the science requires them.
- Show longitude and latitude information so the map remains geographically locatable.
- Handle `0/360` or `-180/180` transitions so longitude display remains continuous.

### 3.2 Projection selection

| Region | Recommended projection | Notes |
|---|---|---|
| Global | `PlateCarree` or `Robinson` | `Robinson` for visual appeal; `PlateCarree` for direct coordinate reading |
| Tropics (30°S–30°N) | `PlateCarree` or `Mercator` | Minimal distortion in tropical belt |
| Mid-latitude regional (e.g., East Asia, Europe, CONUS) | `LambertConformal` | Set `central_longitude` and `standard_parallels` to region center |
| Polar / high-latitude | `NorthPolarStereo` or `SouthPolarStereo` | Set appropriate `true_scale_latitude` |
| Single hemisphere | `Orthographic` | Good for conceptual/overview figures |
| Cross-basin (e.g., Pacific-centered) | `PlateCarree(central_longitude=180)` | Avoids splitting the Pacific |

### 3.3 Field and overlay readability

- Smooth scalar fields use continuous shading (`contourf`).
- For vector fields: choose an appropriate reference arrow length that makes the dominant signal clearly readable. Avoid arrows that are too sparse or too crowded; typical grid skip: every 3–5 points for 1°, every 2–3 for 2.5°, adjust for resolution.
- Place the `quiver key` (reference arrow with label and units) in an uncluttered corner outside or at the edge of the plot area.
- When overlaying significance shading or stippling on a filled field, use light-weight markers (small dots at intervals, or sparse hatching with alpha < 0.5) to avoid obscuring the main signal. Prefer stippling (`.` markers) over dense hatching.

### 3.4 Colors and scales

- Every colorbar must include units.
- Use scientifically meaningful ticks; avoid meaningless high-precision decimals.
- Comparable panels should share the same color scale when possible.
- Adjust colorbar range and intervals to the data. Check the actual data min/max and percentiles before choosing fixed ranges.

## 4. Other figure types

### 4.1 General rules

- Keep the same standard of clarity and restraint.
- When colors other than black are needed, use a fresh pastel palette by default.
- Keep axes, units, legends, and colorbars complete and readable.
- Match the plot type to the scientific question; avoid decorative or low-information choices.
- For dense data, reduce clutter with transparency, aggregation, or sampling.

### 4.2 Time series

- Use clear line styles (solid, dashed, dotted) and distinct colors for multiple variables.
- Include a legend when more than one line is plotted.
- Label the x-axis with human-readable date formats; avoid raw timestamp numbers.
- Add a zero line for anomaly time series.

### 4.3 Vertical cross-sections

- Use pressure (hPa) as the y-axis with inverted direction (1000 at bottom, upper levels at top).
- Label the x-axis with latitude, longitude, or distance as appropriate.
- For filled fields, follow the same colormap rules as maps.

### 4.4 Hovmöller diagrams

- Place time on one axis and space (latitude or longitude) on the other.
- Use the same colormap conventions as maps.
- Ensure time labels are readable and the spatial axis is properly labeled.

### 4.5 Scatter / regression plots

- Show the regression line, equation, R², and p-value when a fit is performed.
- Use transparency (alpha 0.3–0.6) for dense point clouds.

## 5. Layout and export

- Keep balanced whitespace. Avoid crowding, excessive emptiness, and unreadable vector densities.
- Align borders, axes, colorbars, and text as much as possible.
- Share legends or colorbars across comparable panels when scientifically appropriate.
- Do not mix severely conflicting visual styles inside one final figure.
- For multi-panel figures, use `constrained_layout=True` or explicit `gridspec` with consistent spacing.
- Export in `PNG` (600 dpi) and `SVG`.

## 6. Quick reject checklist

If **any** of the following occur, revise the figure immediately:

**Layout & text:**
- [ ] A main title appears
- [ ] Irrelevant in-figure labels such as `ERA5 monthly mean` or other source/process stamps
- [ ] Text, ticks, legends, colorbars, or panel labels overlap or are clipped
- [ ] Font size is too small to read at final output size (below 7 pt)

**Labels & annotations:**
- [ ] Axis labels, units, a colorbar, or essential legend is missing
- [ ] A multi-panel figure lacks `(a)`, `(b)`, `(c)` labels
- [ ] Colorbar lacks units

**Colormap & scale:**
- [ ] The colormap is not from `cmaps`, or is applied without discrete levels
- [ ] A perceptually non-uniform colormap (jet, rainbow, viridis) is used without justification
- [ ] An anomaly/difference field uses a non-diverging colormap or is not centered at 0
- [ ] Comparable panels use misleadingly different color scales

**Maps & spatial:**
- [ ] Map projection is inappropriate for the region
- [ ] Coastlines are missing
- [ ] Longitude/latitude information is missing or discontinuous
- [ ] Vector density is too sparse or too dense; reference arrow is missing or unlabeled

**Scientific coherence:**
- [ ] Significance stippling obscures the main signal
- [ ] The figure is technically visible but not coherent or journal-grade
- [ ] Contour lines are added redundantly on top of a filled contour without adding information
