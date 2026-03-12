# Plot Standards

Use this file for any figure rendering or any `RR` step.
Judge the rendered figure, not just the code.
Before declaring `PASS`, open the actual exported `PNG`, `SVG`, or `PDF`. If needed, inspect a rendered preview for `SVG` or `PDF`. Do not judge from source code, notebook cells, or raw `SVG`/`PDF` text alone.

## Quick map

- `Core defaults`
- `Maps`
- `Other figure types`
- `Layout and export`
- `Quick reject checklist`

## 1. Core defaults

- Aim for clear, publication-grade atmospheric-science figures.
- Do not place a large title inside the figure by default.
- Keep explanatory text in the caption or `README.md`; keep only necessary information inside the figure.
- Combine comparable panels into one final file. Use panel labels `(a)`, `(b)`, `(c)` with a short panel subtitle when needed.
- Keep the visual language clean, balanced, and readable.
- Do not allow text, ticks, legends, colorbars, or panel labels to overlap or clip.
- Use a white background and restrained styling. Do not use 3D effects, shadows, decorative gradients, or patterned fills.
- Use the discrete color scale from the `cmaps` package as the colormap by default, and choose the colormap family, numeric range, and interval to match the variable and scientific question rather than plotting defaults.
- Reuse the project's font setup if it already exists; otherwise use one common sans-serif family consistently.
- Use English in-figure text by default for journal-style outputs unless the user or project requires otherwise.

Text, axis labels, legends, and colorbars must remain readable at final output size.

## 2. Maps

### 2.1 Geospatial setup

- Draw `coastlines` by default. Do not draw national borders unless the user asks or the science requires them.
- Match the projection to the region and task.
- Show longitude and latitude information so the map remains geographically locatable. Data, coastlines, gridlines, and projection must align.
- Handle `0/360` or `-180/180` transitions so longitude display remains continuous.

### 2.2 Field and overlay readability

- Smooth scalar fields may use continuous shading, but the color scale and ticks must stay interpretable.
- For vector fields, choose an appropriate reference arrow and avoid arrows that are too sparse or too crowded.
- Make the `quiver key` or vector scale clear.
- Keep significance shading, hatching, or stippling from obscuring the main signal.

### 2.3 Colors and scales

- Use scientifically standard colormaps. If the project already has a domain convention, keep it.
- For map colorbars, use the `cmaps` package by default, preferably with discrete color coordinates or levels. 
- Choose colormap style, range, and interval scientifically for the variable and comparison target rather than relying on plotting defaults.
- Every colorbar must include units.
- Use scientifically meaningful ticks; avoid meaningless high-precision decimals.
- Comparable panels should share the same color scale when possible.
- If compared quantities have different units or intrinsic scales, separate colorbars are allowed but must be labeled clearly.
- For anomaly, bias, or difference fields, prefer a diverging colormap with white centered at `0`.
- Use symmetric ranges for centered anomaly fields when appropriate.
- Adjust colorbar range and intervals to the data.

## 3. Other figure types

- Keep the same standard of clarity and restraint.
- Use a restrained qualitative palette when no field-specific color convention applies.
- Keep axes, units, legends, and colorbars complete and readable.
- Match the plot type to the scientific question; avoid decorative or low-information choices.
- For dense data, reduce clutter with transparency, aggregation, or sampling.
- Add contours, reference lines, uncertainty bands, or regressions only when they are scientifically useful.

## 4. Layout and export

- Keep balanced whitespace. Avoid crowding, excessive emptiness, and unreadable vector densities.
- Align borders, axes, colorbars, and text as much as possible.
- Share legends or colorbars across comparable panels when scientifically appropriate.
- Do not mix severely conflicting visual styles inside one final figure.
- Export in two tiers:
  - preview: fast `PNG`
  - delivery: high-quality `PNG` plus `SVG`
- Increase DPI only at final delivery; do not use heavy export settings for every draft.
- In `RR` or final handoff, inspect at least one real preview export and the delivery export when both exist.

## 5. Quick reject checklist

If any of the following occur, revise the figure:
- a large in-figure title dominates the space
- text, ticks, legends, colorbars, or panel labels overlap or are clipped
- axis labels, units, a colorbar, or essential legend explanation is missing
- a multi-panel figure lacks `(a)`, `(b)`, and similar labels
- vector density is too sparse or too dense
- comparable panels use misleadingly different color scales
- map projection, boundary handling, or geolocation cues are inconsistent with the task
- the actual exported figure was never opened and checked
- the figure is technically visible but still not coherent or journal-grade
