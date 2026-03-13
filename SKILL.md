---
name: atmospheric-science-research
description: >-
  Use this skill for file-based atmospheric or climate work needing a project
  structure. Trigger for NetCDF, GRIB, reanalysis (ERA5, CMIP, WRF), satellite
  data, or model output; for diagnostics such as EOF, climatology, anomaly,
  trend, regression, correlation, composite, teleconnection, cross-section, or
  Hovmöller; or for revising existing atmospheric figures (labels, colorbars,
  legends, panel layout). Do not use for forecasts, literature summaries, pure
  writing, generic plotting, app or UI work, deployment, or simple downloads.
---

# Atmospheric Science Research

## Modes

- `Full`: `Plan -> Execute -> Validate -> RR`
- `Execute-lite`: `Execute -> Validate -> RR`
- `RR-only`: `RR`

Choose a mode:
- `Full` — new project, new dataset, new method, or anything that may change the scientific conclusion
- `Execute-lite` — constrained patch (labels, ticks, colors, levels, spacing, export) then rerender
- `RR-only` — inspect or revise existing figures
- unclear boundary → `Full`

## Reference loader

Load only what the current step needs — never all at once:
- [references/plan.md](references/plan.md) — planning
- [references/compute-acceleration.md](references/compute-acceleration.md) — compute, diagnostics, data validation
- [references/plot-standards.md](references/plot-standards.md) — figure rendering, colormap, layout
- [references/review.md](references/review.md) — RR loop, physical-sense checks
- [references/readme-template.md](references/readme-template.md) — README

## 1. Project structure

Reuse the project's existing layout. Only fall back to `data_processed/` + `figN/` when no layout exists. Default structure:
- `data_processed/*.nc` — reusable intermediates
- `fig1/plot_*.py` — plotting scripts
- `fig1/compute_*.py` — optional preprocessing
- `fig1/fig1_*.png`, `fig1/fig1_*.svg` — figure outputs

## 2. Plan

Load [plan.md](references/plan.md) + [readme-template.md](references/readme-template.md). Inspect data only enough to design the work. Ask the user if scientific ambiguity remains. Write a short Chinese `README.md`.

## 3. Execute

Load [compute-acceleration.md](references/compute-acceleration.md) for computation, [plot-standards.md](references/plot-standards.md) for figures.

Split compute and plot when subagents are available:
- `compute agent` — ingest, validate, preprocess, save intermediates (NetCDF with units and attributes)
- `plot agent` — read intermediates, render, apply plot standards

Otherwise keep the split in separate scripts. Match existing project style when modifying.

## 4. Validate

Before `RR`, verify outputs:
- **Data**: reopen saved `.nc`; confirm dimensions, coordinates, units, value range
- **Figures**: confirm `.png` and `.svg` exist and are non-empty
- Fix and re-run if anything fails.

## 5. RR

Load [review.md](references/review.md) + [plot-standards.md](references/plot-standards.md). Open the actual `PNG`, check against standards and physical sense, patch minimum code, repeat until `PASS` or `BLOCKED`.

## 6. README

One `README.md` per project. Write the plan version in `Full` after data inspection. Update at final handoff if paths, outputs, or image summary changed. For later modifications, append to `后续版本补充` only.

## 7. Rules

- Search the web only after local docs fail to resolve formulas, methods, or package usage.
- When uncertainty remains, stop and ask the user.
- On failure: read traceback, check common causes (path, coordinate name, dtype), do not retry the same command more than twice, surface the issue if stuck.
- Do not force the full flow for a tiny patch, create extra planning documents, or rewrite working compute code for a figure-only issue.
