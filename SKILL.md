---
name: atmospheric-science-research-skill
description: >-
  Use this skill for file-based atmospheric or climate work that needs a real
  project structure rather than a one-off answer. Trigger it for NetCDF, GRIB,
  reanalysis (ERA5, JRA-55, MERRA-2, NCEP/NCAR), model output (CMIP, WRF, GFS),
  station observations, or existing atmospheric figure files when the task
  involves diagnostics or statistics, regridding, reusable computation,
  compute-vs-plot separation, publication-grade figures, or review of actual
  outputs. Also use it for small figure-only patches inside an existing
  atmospheric project when labels, levels, colorbars, legends, panel letters,
  layout, or exports need fixing without changing the scientific method.
  Make sure to use this skill whenever the user mentions atmospheric data
  processing, climate diagnostics, meteorological visualization, EOF/SVD
  analysis, composite analysis, or wants to create research-quality figures from
  atmospheric or climate datasets, even if they don't explicitly mention
  "research" or "publication". Do not use it for forecasts, literature summaries,
  pure writing, generic plotting outside atmospheric science, app or UI work,
  deployment, or simple one-step downloads.
---

# Atmospheric Science Research

## Think Before Coding

**Surface assumptions. Clarify ambiguity. Don't guess silently.**

Before writing any code, state what you assume about the data — units, coordinate convention (0–360 vs −180/180), temporal resolution, variable naming, and the target climatology baseline. If multiple valid interpretations exist (e.g., "annual mean" could be calendar-year or DJF-anchored; "anomaly" needs a reference period), **ask the user** rather than picking one silently.

## Core Standards

These rules exist because atmospheric science figures follow strict domain conventions — violating them produces scientifically misleading or unpublishable output.

1. **Colormaps from `cmaps` with discrete levels** — Domain-specific colormaps carry scientific meaning (blue-white-red for anomalies, brown-to-green for precipitation). Matplotlib defaults like `jet` create artificial visual boundaries that distort data perception. Always: `import cmaps` + explicit `levels`. Never: `jet`, `rainbow`, `viridis`, or any matplotlib built-in.

2. **No titles or stamps in figures** — In publications, titles waste space and duplicate the figure caption. Source labels ("ERA5 monthly mean") clutter the figure. All metadata belongs in the caption, not `suptitle()`.

3. **Review the rendered PNG, not just the code** — Code that looks correct can render with overlapping labels, clipped elements, or wrong colors. The visual output is the ground truth — open the actual PNG with the Read tool every RR iteration.

4. **Reuse the project's existing layout** — Existing directories represent the user's organizational decisions. Only create `data_processed/` + `figN/` when no project structure exists.

5. **Separate compute from plot** — Atmospheric computations (regridding, EOF, climatology) on large datasets are expensive. Separation allows re-rendering figures without recomputing. Figure-only patches must never touch compute code.

6. **No environment checks** — All packages are pre-installed. Running `pip install`, `conda install`, import-checks, or version-probes wastes time and risks errors. Start coding directly.

7. **Load [plot-standards.md](references/plot-standards.md) before writing plot code** — it contains colormap tables, vector specs, and the quick reject checklist.

## Workflow

Understand the data first (variables, dims, coords, units), then execute. Pick initial contour levels from domain knowledge — refine visually in the RR loop, not by profiling beforehand.

| Situation | Flow |
|---|---|
| New or modified compute/plot | **Execute → RR → Doc** |
| Figure-only fix (labels, ticks, colors, spacing, DPI, export) | **Patch → RR → Doc** |
| Review / QC an existing rendered figure | **RR only** |

---

## Execute

Split into compute + plot. Use subagents when available — agree on output path and variable names first; plot waits if compute fails. Fix and re-run on failure; do NOT enter RR with broken outputs.

### Compute

- Save intermediates to `data_processed/*.nc` with `units` and `long_name` attributes.
- **Prefer fast tools and algorithms**: use `cdo` for regridding/climatology/temporal statistics when it can replace multi-step xarray code; use vectorized numpy/scipy or `apply_ufunc(dask="parallelized")` instead of Python loops; subset region/time/level before heavy ops; stay lazy — delay `.compute()` until the final step.

### Plot — load [plot-standards.md](references/plot-standards.md) first

Read from saved intermediates, not raw data.

- **Colormap**: `import cmaps`, discrete `levels`. Absolute → sequential; anomaly/difference → diverging, symmetric, center 0. Comparable panels share cmap + range.

  Quick lookup (full table in [plot-standards.md](references/plot-standards.md)):

  | Variable | Absolute | Anomaly / Difference |
  |---|---|---|
  | Temperature | `temp_19lev`, `hotcolr_19lev` | `temp_diff_18lev`, `BlueWhiteOrangeRed` |
  | Precipitation | `precip_11lev`, `precip3_16lev` | `precip_diff_12lev`, `CBR_drywet` |
  | SST | `temp_19lev`, `cmocean_thermal` | `GHRSST_anomaly`, `MPL_sstanom` |
  | Geopotential | `BlAqGrYeOrRe`, `BlAqGrYeOrReVi200` | — |
  | SLP | `MPL_YlOrRd`, `BlAqGrYeOrRe` | — |
  | Generic | `BlAqGrYeOrReVi200`, `MPL_YlOrRd` | `BlueWhiteOrangeRed`, `BlWhRe` |

- **Panel labels**: `ax.text(-0.02, 1.03, "(a) subtitle", transform=ax.transAxes, fontsize=11, fontweight="bold", va="bottom", ha="right")`.
- **Colorbar**: show units. Font readable at print size, min 7 pt.
- **Contour lines**: do NOT add on `contourf` unless they convey extra information (e.g., geopotential contours over temperature fill).
- **Vectors**: skip by resolution (1°→3–5, 0.25°→8–15, 2.5°→1–2; larger domain → more skip). Quiver key: reference arrow **outside axes at top-left, aligned with panel labels**; ref magnitude = round number near **median** speed. Full spec in [plot-standards.md](references/plot-standards.md).
- **Stippling**: sparse enough to identify significant regions without obscuring fill. Hatching `"..."` or scatter `[::3]`, size 0.3–1.0, alpha 0.4–0.6. Full spec in [plot-standards.md](references/plot-standards.md).
- **Export**: PNG 600 dpi + SVG, `bbox_inches="tight"`.

### Patch (figure-only fix)

1. Read existing code. Identify the MINIMUM change needed.
2. Patch ONLY affected code — no directory restructuring, no compute rewrite.
3. Re-render affected figures only.

---

## RR — Review & Revision Loop

Load [review.md](references/review.md) + [plot-standards.md](references/plot-standards.md) before starting. Follow every check listed.

**Loop until PASS or BLOCKED.**

Each iteration:
1. Open actual PNG with Read tool.
2. Check against Core Standards + quick reject checklist ([plot-standards.md](references/plot-standards.md)).
3. Sanity glance — are values physically plausible?
   - Temperature should be in K or °C, not raw integers or implausible ranges.
   - Precipitation should be non-negative; wind speed should be non-negative.
   - Cyclonic vorticity sign should match the hemisphere (negative in NH for ζ).
   - Spatial patterns should make physical sense (warm SST in tropics, westerlies in mid-latitudes).
4. Classify:
   - Visual defect → **REVISE** plot code, re-render → step 1.
   - Data/diagnostic issue → **REVISE** compute + replot → step 1.
   - Science question → **BLOCKED**: state clearly, stop.
   - Clean → **PASS**, stop.
5. Same defect after 2 fixes → recheck data, coords, units, dtypes from scratch.
6. Max 10 iterations. If not PASS, surface ALL remaining issues to user.

---

## Documentation — after RR reaches PASS

Load [readme-template.md](references/readme-template.md). Write or update a Chinese `README.md`:
- New project → full README.
- Later modifications → append to「版本更新」, update affected sections only.

Only produce `README.md` — no other documentation files.

---

## Edge Cases

- Preserve original code style when modifying existing code.
- Search the web only after local docs and domain knowledge fail.
- When data has missing values, verify NaN handling (use `skipna=True` or `nanmean` where appropriate).
- For multi-file datasets, verify time continuity and variable consistency across files before merging.
- When working with pressure-level data, check that the vertical coordinate is ordered correctly (ascending or descending) for the analysis at hand.
