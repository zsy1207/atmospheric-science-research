---
name: atmospheric-science-research
description: Atmospheric and climate science workflows that need a runnable project structure. Probe NetCDF/GRIB/HDF/Zarr, compute diagnostics, render publication-grade figures with visual review-revision loops, and write a Chinese project README. Skip conceptual weather questions with no local files.
---

# Atmospheric Science Research

Use whenever the task touches local atmospheric or climate data files and benefits from a reproducible project structure rather than an inline answer.

## Operating Contract

Deliver a project, not a one-off answer. Preserve existing layout and code style unless restructuring is requested. Create `data_processed/` and `figN/` only when no usable layout exists.

Read references on demand:

| Reference | Trigger |
|---|---|
| `references/plot-standards.md` | Before writing or changing plot code |
| `references/review.md` | Before any review-revision (RR) loop |
| `references/readme-template.md` | After all figures `PASS`, before writing README |

## Ask vs Stop

Ask only when a scientific choice changes the result, offering concrete options:

- season definition, baseline period, anomaly sign;
- region / time range, pressure level;
- statistic or test, regridding target;
- unresolved data inconsistency.

Record every adopted scientific assumption in the plan and final README, including user choices and reversible defaults. Do not leave assumptions only in code. Cover season definition, baseline period, anomaly sign, region/time/level, statistic/test, regridding, unit conversion, longitude convention, and masking choices when relevant.

Report `BLOCKED` if:

- a required input, dependency, or shapefile is unavailable with no safe fallback;
- rendered output cannot be opened.

## Workflow

1. **Probe** — Inspect with metadata-only commands, never Python preview scripts.
   - NetCDF: `ncdump -h file.nc`
   - GRIB: `cdo sinfon file.grib`
   - HDF5: `h5ls -r file.h5` or `h5dump -H file.h5`
   - Zarr / other: a metadata-only Python one-liner (`xr.open_zarr(...).info()`), not a preview script.
   - *Fallback*: If `ncdump`, `cdo`, or `h5ls` report `command not found`, fallback to a metadata-only Python one-liner (e.g., using `xarray`, `netCDF4`, or `h5py` to read headers/info).
2. **Plan** — State outputs, adopted scientific assumptions, paths, and script names briefly.
3. **Execute** — Run compute first, then plot. Each script runs with no required runtime args (e.g., `python compute_xxx.py`).
4. **Review** — Render each PNG, open the image, run RR until `PASS` or `BLOCKED`.
5. **Document** — After all figures `PASS`, write or update one Chinese `README.md`, including adopted assumptions and unresolved risks.

## Compute Rules

- **Separation** — compute reads raw → `data_processed/*.nc`; plots read intermediates only, never recompute heavy diagnostics.
- **Metadata** — saved variables include `units` and `long_name`; preserve coordinates.
- **Vectorize** — prefer `xarray`/`numpy` and `cdo`; avoid grid/time Python loops unless data are tiny.
- **Dask** — open with `chunks="auto"`; build lazy graphs first; trigger `.compute()` / `.load()` / NetCDF write only at the end. Never call `.values` on dask arrays — use `.item()` for scalars or `.to_numpy()` for tiny reduced arrays. Don't rechunk without a measured failure.
- **Pre-checks (before computing)** — verify NaN handling, pressure-level ordering, longitude convention, units.
- **Cleanup** — close datasets or use context managers; `gc.collect()` after deleting large objects.

## Plot Rules

Full standards in `references/plot-standards.md`. Required:

- Apply a top-journal visual standard: every figure must be scientifically defensible, visually balanced, uncluttered, and readable at publication scale.
- `import cmaps` + explicit discrete `levels`. Sequential for absolute fields; diverging, symmetric, 0-centered for anomalies.
- Comparable panels share cmap and levels unless units differ.
- Colormap and levels must be visually reasonable and scientifically meaningful: avoid washed-out signals, saturation, false boundaries, or scales that exaggerate noise.
- Wind vectors must have scientifically interpretable reference magnitudes and visually appropriate size/density: arrows should reveal circulation structure without clutter or empty-looking fields.
- Significance stippling/hatching density must be legible but not dominant: it should identify robust regions without obscuring the plotted field.
- Panel labels only: `ax.set_title("(a) Brief subtitle", loc="left")`. No `suptitle`, centered title, or data stamp.
- Mask ≤ 850 hPa fields over the Tibetan Plateau (`~/code/data/map/Tibet/Tibet.shp`).
- Export PNG @ 600 dpi + SVG; `plt.close(fig)`.

## RR Rules

Full procedure in `references/review.md`. Required:

- Open the rendered PNG every iteration — code review alone is forbidden.
- Per iteration: layout first (colorbar proportion, panel balance, overlap, clipping) → cross-check `plot-standards.md` + Reject Conditions → physics check (range, units, sign convention, pressure-axis orientation) → classify `PASS` / `REVISE` / `BLOCKED`.
- Run at least one RR iteration per figure — never declare `PASS` without going through the loop. Max 10 iterations.
- Top-journal gate — revise if the figure would look out of place in a top atmospheric-science journal: cluttered, unbalanced, or inconsistently styled.

## Patch Mode

For figure-only fixes: read existing code, make the smallest safe change, re-render only affected figures, then RR. Do not rewrite compute code or reorganize directories.
