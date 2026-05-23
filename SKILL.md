---
name: atmospheric-science-research
description: Atmospheric and climate science workflows that need a runnable project structure. Probe NetCDF/GRIB/HDF/Zarr, compute diagnostics, render publication-grade figures with visual review-revision loops, and write a Chinese project README. Skip conceptual weather questions with no local files.
---

# Atmospheric Science Research

Use whenever the task touches local atmospheric or climate data files and benefits from a reproducible project structure rather than an inline answer.

## Operating Contract

Read references on demand:

| Reference | Trigger |
|---|---|
| `references/plot-standards.md` | Before writing or changing plot code |
| `references/review.md` | Before any review-revision (RR) loop |
| `references/readme-template.md` | After all figures `PASS`, before writing README |

## Routing

| Request type | Entry point |
|---|---|
| New analysis on local data | Full Workflow (1–7) |
| Figure-only fix on existing project | Patch Mode (skip 1–4, run 5–6) |
| Inspect data without producing figures | Workflow step 1 only |

## Workflow

1. **Probe** — Inspect with metadata-only commands, never Python preview scripts.
   - NetCDF: `ncdump -h file.nc`
   - GRIB: `cdo sinfon file.grib`
   - HDF5: `h5dump -H file.h5`
   - Zarr / other: a metadata-only Python one-liner (`xr.open_zarr(...).info()`), not a preview script.
   - *Fallback*: If `ncdump`, `cdo`, or `h5dump` report `command not found`, fallback to a metadata-only Python one-liner (e.g., using `xarray`, `netCDF4`, or `h5py` to read headers/info).
2. **Plan** — Ask only scientific questions that can change the result. For each question, provide your recommended answer. Record every adopted scientific assumption in the plan and final README, including user choices and reversible defaults. Do not leave assumptions only in code.
3. **Build** — Keep or create a runnable project layout. Prefer `figN/compute_*.py`, `figN/plot_*.py`, and `data_output/figN/...`; do not reorganize existing projects unless requested.
4. **Compute** — Scripts must run without required CLI arguments, be idempotent, and write reusable outputs before plots consume them.
5. **Plot** — Read `plot-standards.md`; apply the journal figure style and code-derived layout conventions there.
6. **Review** — Render each PNG, open the image, run RR until `PASS` or `BLOCKED`.
7. **Document** — After all figures `PASS`, write or update one Chinese `README.md`, including adopted assumptions and unresolved risks.

## Compute Rules

- **Use fast tools** — Use faster, more efficient Python packages such as `cdo`, `dask` and `numpy`, along with optimized algorithms like vectorized operations. Avoid slow loops and inefficient methods.
- **NetCDF (single file)** — `xarray.open_dataset(path, engine="h5netcdf", chunks="auto")`; raw fields stay lazy until NetCDF write.
- **NetCDF (multi file)** — `xarray.open_mfdataset(paths, engine="h5netcdf", parallel=True, chunks="auto")`.
- **Other formats** — For GRIB/HDF/Zarr/other xarray-readable inputs, preserve parallel dask-backed reads with `parallel=True` and `chunks="auto"` or the reader's closest equivalent. Do not rechunk.
- Verify coordinate names, longitude convention, calendars, pressure units/order, missing values, and area weighting before diagnostics.

## Plot Rules

Full standards in `references/plot-standards.md`. Required:

- Apply a top-journal visual standard: every figure must be scientifically defensible, visually balanced, uncluttered, and readable at publication scale. Compact multi-panel layouts, bold panel letters, thin coastlines, dashed gridlines, narrow colorbars, restrained annotations, and red/blue signed significance markers.
- `import cmaps` + explicit discrete `levels`. Sequential for absolute fields; diverging, symmetric, 0-centered for anomalies.
- Comparable panels share cmap and levels unless units differ.
- Colormap and levels must be visually reasonable and scientifically meaningful: avoid washed-out signals, saturation, false boundaries, or scales that exaggerate noise.
- Use Arial for new figures; preserve an existing project font only when patching to avoid style drift. Write units exponentially, e.g. `W m–2`, `kg m–2 s–1`, `m s–1`.
- On maps, show latitude/longitude, add coastlines, and omit national/provincial borders unless scientifically needed.
- Wind vectors must have scientifically interpretable reference magnitudes and visually appropriate size/density: arrows should reveal circulation structure without clutter or empty-looking fields. Sensible density, visible circulation, and a top-right quiver key with a rounded reference magnitude.
- Make significance visible but subordinate: subsampled dots/stippling; positive and negative signs should be distinguishable without obscuring the field.
- Panel labels only: `ax.set_title("a", loc="left")` and brief, necessary descriptions. No `suptitle`, caption, centered title, or data stamp.
- Export PNG @ 600 ppi (`dpi=600`); `plt.close(fig)`.

## RR Rules

Full procedure in `references/review.md`. Required:

- Open the rendered PNG every iteration — code review alone is forbidden.
- Per iteration: layout first (colorbar proportion, panel balance, overlap, clipping) → cross-check `plot-standards.md` + Reject Conditions → physics check (range, units, sign convention, pressure-axis orientation) → classify `PASS` / `REVISE` / `BLOCKED`.
- Run at least one RR iteration per figure — never declare `PASS` without going through the loop. Max 10 iterations; if not `PASS` by then, output `BLOCKED`.
- Top-journal gate — revise if the figure would look out of place in a top atmospheric-science journal: cluttered, unbalanced, or inconsistently styled.

## Patch Mode

For figure-only fixes: read existing code, make the smallest safe change, re-render only affected figures, then RR. Do not rewrite compute code or reorganize directories.
