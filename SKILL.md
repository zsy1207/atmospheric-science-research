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

### Quick signals

| If the request mentions… | → Mode |
|---|---|
| New data / new method / new diagnostic / new figure from scratch | `Full` |
| Add significance test / change diagnostic / switch variable / alter scientific conclusion | `Full` |
| Fix colorbar / label / font / ticks / spacing / DPI / export format | `Execute-lite` |
| Same figure, just swap colormap or adjust levels | `Execute-lite` |
| Review / check / inspect / QC an existing rendered figure | `RR-only` |

## Reference loader

Load only what the current step needs — never all at once:

| Step | Reference |
|---|---|
| Plan | [plan.md](references/plan.md), [readme-template.md](references/readme-template.md) |
| Execute — compute | [compute-acceleration.md](references/compute-acceleration.md) |
| Execute — plot | [plot-standards.md](references/plot-standards.md) |
| Validate | §4 checklist below |
| RR | [review.md](references/review.md), [plot-standards.md](references/plot-standards.md) |

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

Subagent handoff contract:
- Agree on output path (`data_processed/*.nc`) and variable names before starting.
- Compute agent saves with documented dimensions, coordinates, units, and `long_name`.
- Plot agent reads only from the agreed path — no raw data access.
- If compute fails validation, plot waits.

Otherwise keep the split in separate scripts. Match existing project style when modifying.

## 4. Validate

Gate between Execute and RR. Fix and re-run before proceeding.

**Data checks:**
- Reopen every saved `.nc`; confirm it loads without error
- Dimensions and coordinates: correct names, sizes, order
- Units attribute present and physically correct
- Value range plausible (cross-check with review.md physical-sense table)
- NaN fraction reasonable; no unexpected all-NaN slices or time steps
- Time axis decoded correctly (datetime, not raw integers)
- For multi-file outputs, confirm all expected files exist

**Figure checks:**
- `.png` and `.svg` exist and file size > 0
- Open the PNG to verify it renders (use Read tool)
- No Python warnings or tracebacks during render

Fix and re-run if anything fails. Do not proceed to RR with broken outputs.

## 5. RR

Load [review.md](references/review.md) + [plot-standards.md](references/plot-standards.md). Open the actual `PNG`, check against standards and physical sense, patch minimum code, repeat until `PASS` or `BLOCKED`.

## 6. README

One `README.md` per project. Write the plan version in `Full` after data inspection. Update at final handoff if paths, outputs, or image summary changed. For later modifications, append to `后续版本补充` only.

## 7. Rules

- Search the web only after local docs fail to resolve formulas, methods, or package usage.
- When uncertainty remains, stop and ask the user.
- On failure: read traceback, check common causes (path, coordinate name, dtype), do not retry the same command more than twice, surface the issue if stuck.
- Failure escalation: if the same error recurs after 2 targeted fixes, step back — recheck data, coordinate names, units, and dtypes from scratch before a third attempt.
- Coordinate mismatch is the #1 silent-bug source. Always verify `lat`/`latitude`, `lon`/`longitude`, `time` naming and 0–360 vs −180–180 convention match between datasets before merging or comparing.
- Memory: if a compute step is killed or hangs, reduce chunk size or spatial/temporal subset before retrying.
- Do not force the full flow for a tiny patch, create extra planning documents, or rewrite working compute code for a figure-only issue.
- When modifying existing code, preserve the original style (variable naming, import order, comment density).
