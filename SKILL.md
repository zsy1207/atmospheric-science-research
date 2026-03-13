---
name: atmospheric-science-research-skill
description: >-
  Use this skill for file-based atmospheric or climate work that needs a real
  project structure rather than a one-off answer. Trigger it for NetCDF, GRIB,
  reanalysis, model output, or existing atmospheric figure files when the task
  involves planning, diagnostics or statistics, regridding, reusable
  computation, compute-vs-plot separation, publication-grade figures, or review
  of actual outputs. Also use it for small figure-only patches inside an
  existing atmospheric project when labels, levels, colorbars, legends, panel
  letters, layout, or exports need fixing without changing the scientific
  method. Do not use it for forecasts, literature summaries, pure writing,
  generic plotting outside atmospheric science, app or UI work, deployment, or
  simple one-step downloads.
---

# Atmospheric Science Research

## Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State your assumptions explicitly. If uncertain, **ASK**.
- If multiple interpretations exist, present them — don't pick silently.
- If something is unclear, **STOP**. Name what's confusing. Ask.

## HARD RULES — ALWAYS enforce

1. **Colormaps**: MUST use `cmaps` with discrete `levels`. NEVER matplotlib built-ins (jet, rainbow, viridis). Full spec in Plot section.
2. **No main title.** No source/process stamps (e.g., "ERA5 monthly mean") inside the figure.
3. **Review actual PNG**: ALWAYS open rendered PNG with Read tool. NEVER review by code alone.
4. **Reuse existing layout**: Use the project's existing directories. ONLY fall back to `data_processed/` + `figN/` when none exists.
5. **Separate compute from plot**: Different scripts. Figure-only patches do NOT touch compute.
6. **Coordinate normalization**: ALWAYS verify naming (`lat`/`latitude`, `lon`/`longitude`) and convention (0–360 vs −180/180) match BEFORE any merge, comparison, or plotting.

## Workflow

Understand the data (variables, dims, coords, units), then compute and plot immediately. Use domain knowledge for initial levels — adjust in the RR loop, not by profiling beforehand.

| Situation | Flow |
|---|---|
| New or modified compute/plot | **Execute → RR loop → Doc** |
| Figure-only fixes (labels, ticks, colors, spacing, DPI, export) | **Patch → RR loop → Doc** |
| Review / QC an existing rendered figure | **RR loop only** |

---

## Execute

Split into compute + plot. Use subagents when available — agree on output path and variable names first; plot waits if compute fails.

Fix and re-run if anything fails. Do NOT proceed to RR with broken outputs.

### Compute — MUST read and follow [compute.md](references/compute-standards.md)

- Normalize coordinates first (rename `latitude`→`lat`, handle lon convention, ensure lat south-to-north).
- Save intermediates to `data_processed/*.nc` with `units` and `long_name` attributes.

### Plot — MUST read and follow [standards.md](references/plot-standards.md)

- Read from saved intermediates, NOT raw data.
- Colormap: MUST `import cmaps`, discrete `levels`, NEVER matplotlib built-ins or continuous interpolation. Absolute fields → sequential; anomaly/difference → diverging, symmetric, centered at 0; comparable panels MUST share cmap and range. By variable — temperature: `temp_19lev`, `hotcolr_19lev` (abs), `temp_diff_18lev`, `BlueWhiteOrangeRed`, `BlWhRe` (anom); precipitation: `precip_11lev`, `precip3_16lev` (abs), `precip_diff_12lev`, `CBR_drywet`, `GMT_drywet` (anom); SST: `temp_19lev`, `cmocean_thermal` (abs), `GHRSST_anomaly`, `MPL_sstanom` (anom); geopotential: `BlAqGrYeOrRe`, `BlAqGrYeOrReVi200`; SLP: `MPL_YlOrRd`, `BlAqGrYeOrRe`; vertical velocity: `BlWhRe`, `NCV_blu_red`; generic anomaly: `BlueWhiteOrangeRed`, `BlWhRe`, `NCV_blu_red`. Full table in [standards.md](references/plot-standards.md).
- Multi-panel labels: `ax.text(-0.02, 1.03, "(a) subtitle", transform=ax.transAxes, fontsize=11, fontweight="bold", va="bottom", ha="right")`.
- Every colorbar MUST show units. Font: Arial, 10–12 pt (single) / 8–10 pt (multi), min 7 pt.
- Do NOT add contour lines on `contourf` unless they convey extra info (e.g., geopotential contours over temperature fill).
- Vectors: density must show spatial structure without clutter — skip by resolution (1° → 3–5 pts, 0.25° → 8–15 pts, 2.5° → 1–2 pts; larger domain needs more skipping). Quiver key **inside lower-right of axes** with semi-transparent background box; reference magnitude = round number near **median** speed, not max (e.g., typical 5–15 m/s → use `10 m/s`).
- Significance stippling: sparse enough to identify significant regions without obscuring the fill. Hatching `"..."` or scatter `[::3]`, marker size 0.3–1.0, alpha 0.4–0.6. High-res data (< 0.5°) must increase subsampling to avoid solid-black patches.
- Export PNG (600 dpi) + SVG, `bbox_inches="tight"`.

### Patch (figure-only fixes)

1. Read existing code. Identify the MINIMUM changes needed.
2. Patch ONLY affected code. Do NOT restructure directories or rewrite compute.
3. Re-render affected figures only.

---

## RR — Review & Revision LOOP

MUST read and follow [review.md](references/review.md) + [standards.md](references/plot-standards.md) before starting.

**This is a loop. Do NOT stop until PASS or BLOCKED.**

Each iteration:
1. Open actual PNG with Read tool.
2. Check against HARD RULES and [standards.md](references/plot-standards.md) quick reject checklist.
3. Sanity glance: values, units, patterns physically plausible?
4. Classify:
   - Visual defect → **REVISE**: patch plot, re-render, **→ step 1**.
   - Data/diagnostic issue → **REVISE**: patch compute + replot, **→ step 1**.
   - Science question → **BLOCKED**: state the question clearly. Stop.
   - Clean → **PASS**. Stop.
5. Same defect after 2 fixes → recheck data/coords/units/dtypes from scratch, then re-enter.
6. Max 5 iterations. If not PASS, surface ALL remaining issues to user.

---

## Documentation — after RR reaches PASS

MUST read and follow [readme-template.md](references/readme-template.md). Write or update a Chinese `README.md`. New project: full README. Later modifications: append to「版本更新」, update affected sections only. ONLY produce `README.md` — no other documentation files.

---

## Failure rules

- Same error after 2 targeted fixes → recheck data, coordinates, units, and dtypes from scratch.
- Preserve original code style when modifying existing code.
- Search the web only after local docs fail.
