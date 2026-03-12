# Plot Standards

Use this file for any figure rendering or any `RR` step.

## 1. Core defaults

- Aim for publication-quality figures at the highest standard in atmospheric science.
- Keep in-figure text to the minimum needed to read the science.
- the image created should be professional, scientific, and informative, while maintaining an attractive visual style.
- Do not use a main title.
- Use subfigure labels (a), (b), and (c), with brief subtitles placed just outside the upper-left corner of each subfigure.
- Do not allow text, ticks, legends, colorbars, or panel labels to overlap or clip.
- Use a white background and restrained styling. Do not use 3D effects, shadows, decorative gradients, or patterned fills.
- When a colormap is used, it must come from the `cmaps` package and must be applied with discrete coordinates or levels. Choose the colormap, numeric range, and interval from the variable and science, not plotting defaults.
- Use 'Arial' as the default font.
- Use English in-figure text by default for journal-style outputs unless the user or project requires otherwise.
- Do not place a large title or a source/process stamp inside the figure. Labels such as `ERA5 monthly mean` belong in the caption or `README.md`, not in the graph. Keep explanatory text in the caption or `README.md`.
- Text, axis labels, legends, and colorbars must remain readable at final output size.
- Keep the image clean and clear. Do not add any meaningless labels or text.If a variable is shown with a filled contour plot, do not add contour lines.


## 2. Maps

### 2.1 Geospatial setup

- Draw `coastlines` by default. Do not draw national borders unless the user asks or the science requires them.
- Match the projection to the region and task.
- Show longitude and latitude information so the map remains geographically locatable.
- Handle `0/360` or `-180/180` transitions so longitude display remains continuous.

### 2.2 Field and overlay readability

- Smooth scalar fields use continuous shading.
- For vector fields, choose an appropriate reference arrow and avoid arrows that are too sparse or too crowded.
- Make the `quiver key` or vector scale clear.
- Keep significance shading, hatching, or stippling from obscuring the main signal.

### 2.3 Colors and scales

- For map colorbars, you must use a `cmaps` colormap with discrete coordinates or levels. 
- Within `cmaps`, choose the commonly used colormap that matches the variable(don't use 'MPL_viridis'). Choose colormap style, range, and interval scientifically for the variable and comparison target rather than relying on plotting defaults.
- Every colorbar must include units.
- Use scientifically meaningful ticks; avoid meaningless high-precision decimals.
- Comparable panels should share the same color scale when possible.
- If compared quantities have different units or intrinsic scales, separate colorbars are allowed.
- For anomaly, bias, or difference fields, prefer a diverging colormap with white centered at `0`. Use symmetric ranges for centered anomaly fields when appropriate.
- Adjust colorbar range and intervals to the data.

## 3. Other figure types

- Keep the same standard of clarity and restraint.
- When colors other than black are needed, use a fresh pastel palette by default.
- Keep axes, units, legends, and colorbars complete and readable.
- Match the plot type to the scientific question; avoid decorative or low-information choices.
- For dense data, reduce clutter with transparency, aggregation, or sampling.

## 4. Layout and export

- Keep balanced whitespace. Avoid crowding, excessive emptiness, and unreadable vector densities.
- Align borders, axes, colorbars, and text as much as possible.
- Share legends or colorbars across comparable panels when scientifically appropriate.
- Do not mix severely conflicting visual styles inside one final figure.
- Export in `PNG`(600dpi) and `SVG`

## 5. Quick reject checklist

If any of the following occur, revise the figure:
- a main title.
- irrelevant in-figure labels such as `ERA5 monthly mean` or other source/process stamps appear
- text, ticks, legends, colorbars, or panel labels overlap or are clipped
- axis labels, units, a colorbar, or essential legend explanation is missing
- a multi-panel figure lacks `(a)`, `(b)`, and similar labels
- the continuous colormap or discrete color scale do not satisfy the required specifications.
- vector density is too sparse or too dense
- comparable panels use misleadingly different color scales
- map projection, boundary handling, or geolocation cues are inconsistent with the task
- the figure is technically visible but still not coherent or journal-grade
