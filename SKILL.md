---
name: atmospheric-science-research-skill
description: >-
  Use this skill for file-based atmospheric or climate work that needs a real
  project structure rather than a one-off answer. Trigger it for NetCDF, GRIB,
  reanalysis, model output, or existing atmospheric figure files when the task
  involves diagnostics or statistics, regridding, reusable computation,
  compute-vs-plot separation, publication-grade figures, or review of actual
  outputs. Also use it for small figure-only patches inside an existing
  atmospheric project when labels, levels, colorbars, legends, panel letters,
  layout, or exports need fixing without changing the scientific method.
  Do not use it for forecasts, literature summaries, pure writing, generic
  plotting outside atmospheric science, app or UI work, deployment, or simple
  one-step downloads.
---

# Atmospheric Science Research

## Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**
State assumptions explicitly. If uncertain or multiple interpretations exist — **ASK**, don't pick silently.

## HARD RULES — ALWAYS enforce

1. **Colormaps**: `import cmaps` + discrete `levels`. NEVER matplotlib built-ins (`jet`, `rainbow`, `viridis`).
2. **No title**: no `suptitle()` or source/data stamps (e.g. "ERA5 monthly mean") inside the figure.
3. **Review actual PNG**: open the rendered PNG with Read tool every RR iteration. NEVER judge by code alone.
4. **Reuse existing layout**: use the project's own directories. Fall back to `data_processed/` + `figN/` only when none exist.
5. **Separate compute from plot**: different scripts. Figure-only patches never touch compute.
6. **Coordinate normalization**: verify naming (`lat`/`latitude`, `lon`/`longitude`) and convention (0–360 vs −180/180) BEFORE any merge, comparison, or plot.

## Workflow

Understand the data first (variables, dims, coords, units), then execute. Pick initial levels from domain knowledge — refine in the RR loop, not by profiling beforehand.

| Situation | Flow |
|---|---|
| New or modified compute/plot | **Execute → RR → Doc** |
| Figure-only fix (labels, ticks, colors, spacing, DPI, export) | **Patch → RR → Doc** |
| Review / QC an existing rendered figure | **RR only** |

---

## Execute

Split into compute + plot. Use subagents when available — agree on output path and variable names first; plot waits if compute fails. Fix and re-run on failure; do NOT enter RR with broken outputs.

### Compute — read [compute-standards.md](references/compute-standards.md) first

- Normalize coordinates first (rename `latitude`→`lat`, handle lon convention, ensure lat south→north).
- Save intermediates to `data_processed/*.nc` with `units` and `long_name` attributes.

### Plot — read [plot-standards.md](references/plot-standards.md) first

Read from saved intermediates, not raw data.

- **Colormap**: `import cmaps`, discrete `levels`, NEVER matplotlib built-ins. Absolute → sequential; anomaly/difference → diverging, symmetric, center 0. Comparable panels share cmap + range.

  Quick lookup (full table in [plot-standards.md](references/plot-standards.md)):

  | Variable | Absolute | Anomaly / Difference |
  |---|---|---|
  | Temperature | `temp_19lev`, `hotcolr_19lev` | `temp_diff_18lev`, `BlueWhiteOrangeRed`, `BlWhRe` |
  | Precipitation | `precip_11lev`, `precip3_16lev` | `precip_diff_12lev`, `CBR_drywet`, `GMT_drywet` |
  | SST | `temp_19lev`, `cmocean_thermal` | `GHRSST_anomaly`, `MPL_sstanom` |
  | Geopotential | `BlAqGrYeOrRe`, `BlAqGrYeOrReVi200` | — |
  | SLP | `MPL_YlOrRd`, `BlAqGrYeOrRe` | — |
  | Vert. velocity | — | `BlWhRe`, `NCV_blu_red` |
  | Generic anomaly | — | `BlueWhiteOrangeRed`, `BlWhRe`, `NCV_blu_red` |

- **Panel labels**: `ax.text(-0.02, 1.03, "(a) subtitle", transform=ax.transAxes, fontsize=11, fontweight="bold", va="bottom", ha="right")`.
- **Colorbar**: must show units. Font: DejaVu Sans, 10–12 pt (single) / 8–10 pt (multi), min 7 pt.
- **Contour lines**: do NOT add on `contourf` unless they convey extra information (e.g. geopotential contours over temperature fill).
- **Vectors**: skip by resolution (1°→3–5, 0.25°→8–15, 2.5°→1–2; larger domain → more skip). Quiver key: arrow + label in **white opaque box flush against lower-right border**; arrow sized to fit inside box; ref magnitude = round number near **median** speed, not max. Full spec in [plot-standards.md](references/plot-standards.md).
- **Stippling**: sparse enough to identify significant regions without obscuring fill. Hatching `"..."` or scatter `[::3]`, size 0.3–1.0, alpha 0.4–0.6. High-res (< 0.5°) needs more subsampling. Full spec in [plot-standards.md](references/plot-standards.md).
- **Export**: PNG 600 dpi + SVG, `bbox_inches="tight"`.

### Patch (figure-only fix)

1. Read existing code. Identify the MINIMUM change needed.
2. Patch ONLY affected code — no directory restructuring, no compute rewrite.
3. Re-render affected figures only.

---

## RR — Review & Revision Loop

Read [review.md](references/review.md) + [plot-standards.md](references/plot-standards.md) before starting.

**Loop until PASS or BLOCKED.**

Each iteration:
1. Open actual PNG with Read tool.
2. Check against HARD RULES + quick reject checklist ([plot-standards.md](references/plot-standards.md)).
3. Sanity glance: values, units, spatial patterns physically plausible?
4. Classify:
   - Visual defect → **REVISE** plot code, re-render → step 1.
   - Data/diagnostic issue → **REVISE** compute + replot → step 1.
   - Science question → **BLOCKED**: state clearly, stop.
   - Clean → **PASS**, stop.
5. Same defect after 2 fixes → recheck data, coords, units, dtypes from scratch.
6. Max 10 iterations. If not PASS, surface ALL remaining issues to user.

---

## Documentation — after RR reaches PASS

Read [readme-template.md](references/readme-template.md). Write or update a Chinese `README.md`:
- New project → full README.
- Later modifications → append to「版本更新」, update affected sections only.

Only produce `README.md` — no other documentation files.

---

## Failure Rules

- Same error after 2 targeted fixes → recheck data, coordinates, units, dtypes from scratch.
- Preserve original code style when modifying existing code.
- Search the web only after local docs fail.
