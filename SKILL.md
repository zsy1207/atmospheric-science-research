---
name: atmospheric-science-research
description: >-
  Use this skill for file-based atmospheric or climate work that needs a real
  project structure rather than a one-off answer. Trigger it for NetCDF, GRIB,
  reanalysis (ERA5, JRA-55, MERRA-2), model output (CMIP, WRF, GFS), station
  observations, or existing atmospheric figure files when the task involves
  diagnostics or statistics, regridding, reusable computation,
  compute-vs-plot separation, publication-grade figures, or review of actual
  outputs. Also use it for small figure-only patches inside an existing
  atmospheric project when labels, levels, colorbars, legends, panel letters,
  layout, or exports need fixing without changing the scientific method.
  Do not use it for forecasts, literature summaries, pure writing, generic
  plotting outside atmospheric science, app or UI work, deployment, or
  simple one-step downloads.
---

# Atmospheric Science Research

> **Reference files** — read on demand, not upfront:
> | File | Contents | When to read |
> |---|---|---|
> | [plot-standards.md](references/plot-standards.md) | Colormap tables, vector specs, projections, figure sizing, quick reject checklist | Before writing any plot code |
> | [review.md](references/review.md) | RR loop procedure, quick fix recipes, sanity checks | Before starting an RR loop |
> | [readme-template.md](references/readme-template.md) | Chinese README template | After RR reaches PASS |

## Before Writing Code

Surface assumptions about the data — units, coordinate convention (0–360 vs −180/180), temporal resolution, variable naming, and climatology baseline. If multiple valid interpretations exist (e.g., "annual mean" could be calendar-year or DJF-anchored; "anomaly" needs a reference period), **ask the user** rather than picking one silently.

## Core Standards

These rules reflect atmospheric science domain conventions — violating them produces misleading or unpublishable output.

1. **Colormaps from `cmaps` with discrete levels** — Domain-specific colormaps carry scientific meaning (blue-white-red for anomalies, brown-to-green for precipitation). Always: `import cmaps` + explicit `levels`. Never: `jet`, `rainbow`, `viridis`, or any matplotlib built-in.

2. **No titles or stamps** — In publications, titles duplicate the caption. Source labels clutter the figure. All metadata belongs in the caption, not `suptitle()`.

3. **Review rendered PNG, not just code** — Code that looks correct can render with overlapping labels, clipped elements, or wrong colors. Open the actual PNG with the Read tool in every RR iteration.

4. **Reuse existing project layout** — Existing directories reflect the user's organizational decisions. Only create `data_processed/` + `figN/` when no structure exists.

5. **Separate compute from plot** — Atmospheric computations (regridding, EOF, climatology) are expensive. Separation allows re-rendering without recomputing. Figure-only patches never touch compute code.

6. **No environment setup** — All packages are pre-installed. No `pip install`, `conda install`, import checks, or version probes.

## Workflow

Understand the data first (variables, dims, coords, units), then execute. Pick initial contour levels from domain knowledge — refine visually in the RR loop, not by profiling.

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
- **Prefer fast tools**: `cdo` for regridding/climatology/temporal statistics when it can replace multi-step xarray code; vectorized numpy/scipy or `apply_ufunc(dask="parallelized")` over Python loops; subset region/time/level before heavy ops; stay lazy — delay `.compute()` until the final step.

### Plot — read [plot-standards.md](references/plot-standards.md) first

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
- **Borders**: for national borders use shapefiles from `~/code/data/map/worldmap/world_boundaries.shp`. For China maps use `~/code/data/map/chinamap/simple_china.shp`, and add a South China Sea nine-dash line inset at bottom-right using `~/code/data/map/chinamap/nine_dots.shp`.
- **Export**: PNG 600 dpi + SVG, `bbox_inches="tight"`.

### Patch (figure-only fix)

1. Read existing code. Identify the MINIMUM change needed.
2. Patch ONLY affected code — no directory restructuring, no compute rewrite.
3. Re-render affected figures only.

---

## RR — Review & Revision Loop

Load [review.md](references/review.md) and [plot-standards.md](references/plot-standards.md) before starting. Follow every check listed there.

**Loop until PASS or BLOCKED.** Max 10 iterations per figure.

Each iteration: open PNG → check against Core Standards + quick reject checklist → verify physical plausibility → classify as REVISE / BLOCKED / PASS. Same defect after 2 fixes → recheck data, coords, units, dtypes from scratch.

---

## Documentation — after RR reaches PASS

Load [readme-template.md](references/readme-template.md). Write or update a Chinese `README.md`:
- New project → full README.
- Later modifications → append to「版本更新」, update affected sections only.

Only produce `README.md` — no other documentation files.

---

## Data Handling Gotchas

- Missing values: verify NaN handling (`skipna=True` or `nanmean`).
- Multi-file datasets: verify time continuity and variable consistency before merging.
- Pressure-level data: check vertical coordinate ordering (ascending vs descending).
- Existing code: preserve original style when modifying.
- Web search: use only after local docs and domain knowledge fail.
