---
name: atmospheric-science-research
description: >-
  Handles file-based atmospheric and climate science workflows that need a
  real project structure rather than a one-off answer.
---

# Atmospheric Science Research

> **Reference files** — read on demand, not upfront:
> | File | Contents | When to read |
> |---|---|---|
> | [plot-standards.md](references/plot-standards.md) | Colormap tables, vector specs, projections, figure sizing, quick reject checklist | MUST read before writing ANY plot code |
> | [review.md](references/review.md) | RR loop procedure, quick fix recipes, sanity checks | MUST read before starting ANY RR loop |
> | [readme-template.md](references/readme-template.md) | Chinese README template | MUST read after RR reaches PASS |

## Before Writing Code

**MANDATORY** — inspect inputs and directory structure BEFORE writing a single line of code. Shell probes ONLY — do NOT write Python preview scripts:
- **NetCDF**: `ncdump -h file.nc`
- **GRIB**: `cdo sinfon file.grib`

You MUST confirm: variables, dimensions, coordinates, units, time coverage, spatial region, and missing-data conventions. Writing code without this confirmation is FORBIDDEN — it leads to silent errors and wasted compute.

If multiple valid interpretations exist (e.g., "annual mean" could be calendar-year or DJF-anchored; "anomaly" needs a reference period), you MUST ask the user. NEVER silently pick one — wrong assumptions invalidate entire pipelines.

## Core Standards

**ZERO TOLERANCE** — violating ANY of these rules produces misleading or unpublishable output. Every rule below is a hard constraint, not a suggestion.

1. **`import cmaps` + explicit `levels` — NO EXCEPTIONS.** Domain-specific colormaps carry scientific meaning. NEVER use `jet`, `rainbow`, `viridis`, or any generic matplotlib colormap. These lack the perceptual structure needed for scientific communication and WILL be rejected.

2. **NO titles, NO stamps — EVER.** NEVER call `suptitle()`, `set_title()`, or add source/data stamps to figures. All metadata belongs in the caption. A figure with a title is an immediate REVISE.

3. **ALWAYS open and inspect the rendered PNG — NEVER review code alone.** Code that looks correct can render with overlapping labels, clipped elements, or wrong colors. You MUST open the actual PNG with the Read tool in EVERY RR iteration. Skipping this step is the single most common source of defects.

4. **Reuse existing project layout — do NOT reorganize.** Existing directories reflect the user's decisions. ONLY create `data_processed/` + `figN/` when NO structure exists. Restructuring without explicit permission is FORBIDDEN.

5. **Separate compute from plot — ALWAYS.** Atmospheric computations (regridding, EOF, climatology) are expensive. NEVER mix compute and plot in the same script. Compute saves to disk; plot reads from disk. No exceptions.

6. **NO environment setup — EVER.** All packages are pre-installed. NEVER run `pip install`, `conda install`, import checks, or version probes. These waste time and clutter output.

## Workflow

Understand the data FIRST (variables, dims, coords, units), THEN execute. Pick initial contour levels from domain knowledge — refine visually in RR, NEVER by profiling data ranges in code.

| Situation | Flow |
|---|---|
| New or modified compute/plot | **Execute → RR → Doc** |
| Figure-only fix (labels, ticks, colors, spacing, DPI, export) | **Patch → RR → Doc** |
| Review / QC an existing rendered figure | **RR only** |

---

## Execute

ALWAYS split into compute + plot — no monolithic scripts. Use subagents when available — agree on output path and variable names FIRST; plot waits if compute fails. Fix and re-run on failure; entering RR with broken outputs wastes iterations and is unacceptable.

### Compute

- Save intermediates to `data_processed/*.nc` — MUST include `units` and `long_name` attributes. Missing attributes = incomplete output.
- **Use fast tools** — CDO, vectorized operations, efficient packages. NEVER use nested Python loops or brute-force approaches when a vectorized or CDO-based solution exists.

### Plot — MUST read [plot-standards.md](references/plot-standards.md) first

Read from saved intermediates, NEVER from raw data. Detailed specs (panel labels, colorbar, vectors, stippling, borders, export) are in [plot-standards.md](references/plot-standards.md) — do NOT duplicate here.

- **Colormap**: `import cmaps` + discrete `levels`. Absolute fields → sequential; anomaly/difference → diverging, symmetric, center 0. Comparable panels MUST share colormap and level range — mismatched scales between panels is an immediate REVISE.

  Quick lookup (full table in [plot-standards.md](references/plot-standards.md)):

  | Variable | Absolute | Anomaly / Difference |
  |---|---|---|
  | Temperature | `temp_19lev`, `hotcolr_19lev` | `temp_diff_18lev`, `BlueWhiteOrangeRed` |
  | Precipitation | `precip_11lev`, `precip3_16lev` | `precip_diff_12lev`, `CBR_drywet` |
  | SST | `temp_19lev`, `cmocean_thermal` | `GHRSST_anomaly`, `MPL_sstanom` |
  | Geopotential | `BlAqGrYeOrRe`, `BlAqGrYeOrReVi200` | — |
  | SLP | `MPL_YlOrRd`, `BlAqGrYeOrRe` | — |
  | Generic | `BlAqGrYeOrReVi200`, `MPL_YlOrRd` | `BlueWhiteOrangeRed`, `BlWhRe` |

- **Layout code is NEVER correct on first render** — figsize, subplot arrangement, colorbar size, and element positions are starting-point guesses. You MUST verify proportions in the rendered PNG and refine iteratively in RR. See "Layout Anti-patterns" in [plot-standards.md](references/plot-standards.md).

### Patch (figure-only fix)

1. Read existing code. Identify the MINIMUM change needed — nothing more.
2. Patch ONLY affected code — NO directory restructuring, NO compute rewrite.
3. Re-render affected figures ONLY.

---

## RR — Review & Revision Loop

**MANDATORY**: Load [review.md](references/review.md) and [plot-standards.md](references/plot-standards.md) BEFORE starting. Follow EVERY check listed there — skipping any check is FORBIDDEN.

**Loop until PASS or BLOCKED.** Max 10 iterations per figure. NEVER declare PASS without opening and inspecting the PNG.

Each iteration — this sequence is MANDATORY, do NOT skip or reorder:
1. Open PNG with Read tool
2. Check layout proportions FIRST (colorbar sizing, element crowding, subplot spacing)
3. Check against Core Standards + quick reject checklist — every item
4. Verify physical plausibility (values, units, sign conventions, spatial patterns)
5. Classify: REVISE / BLOCKED / PASS

Same defect after 2 fixes → STOP patching. Recheck data, coords, units, dtypes from scratch — the root cause is upstream.

Layout parameters in code are NEVER correct on first render — the rendered PNG is the **ONLY** ground truth. NEVER trust code-level parameters over what you see in the image.

---

## Documentation — ONLY after RR reaches PASS

MUST load [readme-template.md](references/readme-template.md) first. Write or update a Chinese `README.md`:
- New project → full README.
- Later modifications → append to「版本更新」, update affected sections ONLY.

ONLY produce `README.md` — creating any other documentation file is FORBIDDEN.

---

## Data Handling — Common Traps

These are NOT optional checks — each one has caused silent data corruption in real projects:

- **Missing values**: ALWAYS verify NaN handling (`skipna=True` or `nanmean`). Unhandled NaN = wrong results.
- **Multi-file datasets**: MUST verify time continuity and variable consistency BEFORE merging. NEVER blindly concatenate.
- **Pressure-level data**: MUST check vertical coordinate ordering (ascending vs descending). Wrong ordering = inverted profiles.
- **Existing code**: ALWAYS preserve original style when modifying. Do NOT refactor code you did not write.
- **Web search**: use ONLY after local docs and domain knowledge fail — not as a first resort.
