# Review

`RR` = `review & revision`.
Use this file when figures already exist or the task is mainly to inspect and revise outputs.
Its job is to inspect real figures against `plot-standards.md`, check basic physical sense, validate any changed upstream outputs, and enter an immediate fix-and-rerender loop until the figure passes or a real scientific ambiguity blocks the task.

## RR steps

1. Open the actual output `PNG`. Do not review by code, filename, or directory listing alone. If the figures do not exist yet, render them first.
2. Check them visually against `plot-standards.md`.
   - once a concrete problem is confirmed, do not wait to collect every other issue first
   - move directly into the minimum relevant patch for that problem
3. Run the physical-sense checks (see below).
4. Classify the issue:
   - visual-only -> `REVISE` and patch only the figure-side plotting code, such as `figN/plot_*.py` or the existing project equivalent
   - data or diagnostic issue -> `REVISE` and patch the affected compute code or upstream processed outputs, plus the corresponding plotting code
   - unresolved science -> `BLOCKED`
   - no remaining issue -> `PASS`
5. If the result is `REVISE`, patch immediately and re-render only the affected figure and any directly dependent preview or delivery exports, not the whole set.
6. Re-open the new output and continue issue-by-issue until the result is `PASS` or `BLOCKED`.

## Physical-sense checks

### General checks (all variables)

- [ ] Units and variable type match the axis labels and colorbar
- [ ] Sign convention is correct (e.g., upward flux positive, anomaly sign matches expectation)
- [ ] Value range is physically reasonable for the variable, region, and season
- [ ] No obvious data artifacts (stripes, checkerboard, all-NaN regions that should have data)
- [ ] Land/ocean masking is correct if applied
- [ ] Diagnostics and statistics implied by the processed outputs and compute scripts are reasonable

### Variable-specific checks

| Variable | Key checks |
|---|---|
| Temperature | Absolute T in K (200–320) or °C (−70 to +50); anomaly typically ±0.5–10 K; spatial gradient land > ocean in summer |
| Precipitation | Non-negative; accumulated values match time step; wet regions (ITCZ, storm tracks) in expected locations |
| Wind (u, v, speed) | Speed typically 0–80 m/s; jet stream 30–70 m/s at 200 hPa; surface wind < 25 m/s for climatology |
| Geopotential height | Smooth field; ~5500 m at 500 hPa mid-latitudes; gradient from equator to pole |
| SST | Range −2°C to +35°C for absolute; warm pool, cold tongue in correct locations |
| MSLP / SLP | Range ~960–1050 hPa; low pressure in storm tracks, high pressure in subtropics |
| OLR | Range ~100–300 W/m²; low over deep convection regions, high over deserts |
| Humidity | Relative: 0–100%; Specific: non-negative; moist in tropics, dry in subtropics and over land interior |
| Vorticity | Cyclonic sign matches hemisphere (negative NH, positive SH for relative vorticity) |
| EOF patterns | Leading mode should match known dominant modes (e.g., ENSO for tropical Pacific SST) |

### Vector field checks

- [ ] Arrow direction is physically plausible (e.g., trade winds easterly in tropics, westerlies in mid-latitudes)
- [ ] Arrow magnitude is consistent across the domain (no single wildly large arrow)
- [ ] Reference arrow value and units are labeled

## RR focus

Prioritize these checks:
- text, ticks, legends, and colormap do not overlap or get clipped
- colormap scale ranges and intervals are scientific, and vector density is reasonable
- diagnostic and statistical methods are reasonable
- axis labels, units, and colorbars are complete
- map projection, coastlines, and boundary strategy are correct
- the figure is not crowded, misleading, or visually inconsistent
- the result is clean, readable, and journal-grade
- the exported file actually opens and was checked as a rendered figure, not inferred from code alone

## Non-figure output validation

When compute scripts produce intermediate or processed data files:
- Verify the file was created and is non-empty
- Open it and check dimensions, coordinates, and key variable summaries
- Confirm units and attributes are preserved
- Check for unexpected NaN counts or value ranges

## Iteration limits

- If 3 consecutive `REVISE` rounds fail to fix the same issue, stop and reassess the approach rather than continuing to patch.
- Use `BLOCKED` when the computation method itself is ambiguous or the required change would clearly alter the scientific conclusion.
- Use `PASS` only after the rerendered output has been re-opened and checked.

## README update

After the figure passes, update `README.md`:
- sync the path-structure section if scripts or outputs changed
- update `图片简述` if the figure meaning or appearance changed
- append one concise note in `后续版本补充`
