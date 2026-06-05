---
name: atmospheric-science-research
description: Use for climatology data processing, diagnostic computation, and publication-grade plotting.
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

## Workflow

1. **Probe** — Metadata-only inspection; never write Python preview scripts.
   - NetCDF: `ncdump -h file.nc`
   - GRIB: `cdo sinfon file.grib`
   - HDF5: `h5dump -H file.h5`
   - Zarr / other: metadata-only Python one-liner (`xr.open_zarr(...).info()`).
   - *Fallback*: if a CLI is missing, use a metadata-only Python one-liner (`xarray`, `netCDF4`, or `h5py` to read headers/info).
2. **Plan** — Ask only scientific questions that change the result; provide your recommended answer with each. Record every adopted assumption (user choices and reversible defaults) in the plan and final README, never only in code.
3. **Build** — Keep or create a runnable layout: `figN/compute_*.py`, `figN/plot_*.py`, `data_output/figN/...`. Do not reorganize existing projects unless asked.
4. **Compute** — Scripts run without required CLI args, are idempotent, and write reusable outputs before plots consume them. For long computations, once the output schema is clear, start drafting the plot script while the compute job runs; revise it after outputs exist.
5. **Plot** — Read `plot-standards.md`; apply its journal style and layout conventions.
6. **Review** — Render PNG, open the image, run RR until `PASS` or `BLOCKED`.
7. **Document** — After all figures `PASS`, write or update one Chinese `README.md` listing adopted assumptions and unresolved risks.

## Compute Rules

- **Use fast tools** — Vectorize with `xr.where` / boolean masks, not Python loops over grid points. Reduce with `.mean(dim=..., skipna=True)` (and `.sum` / `.std`) to preserve coords. Climatology and statistics: `groupby("time.dayofyear").mean()`, `xr.corr`, `xr.cov`, `xr.polyfit`, and `xr.apply_ufunc(..., dask="parallelized")` for custom kernels. Regrid with `cdo remap*` (or `xesmf`), not hand-rolled interpolation.
- **NetCDF** — `xarray.open_dataset(path, engine="h5netcdf", chunks="auto")` for one file; `xarray.open_mfdataset(paths, engine="h5netcdf", parallel=True, chunks="auto")` for many. Fall back to the default engine if `h5netcdf` errors. Raw fields stay lazy until NetCDF write.
- **Other formats** — For GRIB / HDF / Zarr / other xarray-readable inputs, preserve parallel dask-backed reads (`parallel=True`, `chunks="auto"`) or the reader's closest equivalent.
- **Dask boundary** — Keep raw daily or sub-daily fields lazy. First reduce/compress them to annual means (or the coarsest scientifically valid temporal stack), then eager-load only that annual-mean stack before algorithmic loops. Build climatologies from the in-memory annual fields; never let a lazy climatology enter a loop.
- **Compute placement** — Do not rechunk. Avoid fine-grained `.compute()` / `.load()` calls inside loops, especially per-year `.compute()`. Trigger dask once at the algorithm boundary where eager-loading prevents repeated I/O, then loop only over in-memory xarray/numpy slices. Do not eager-load raw daily inputs, and do not postpone computation so long that the same lazy graph rereads source data repeatedly.
- **Regional padding** — When computing or saving a regional subset (e.g. `70°E–105°E, 25°N–40°N`), pad the source field by **at least 2 grid points on each side** beyond the target bounds before computing/writing. The plot then sets `set_extent` to the target bounds. Prevents white margins from `contourf` / `pcolormesh` cell-edge handling and interpolation/regrid artifacts at the boundary.

## Plot Rules

Full standards in `references/plot-standards.md`. Required:

- **Top-journal gate** — every figure must be scientifically defensible, visually balanced, uncluttered, and readable at publication scale: compact multi-panel layouts, bold panel letters, thin coastlines, dashed gridlines, narrow colorbars, restrained annotations, signed-significance markers in red/blue.
- `import cmaps` with explicit discrete `levels`. Sequential for absolute fields; diverging, symmetric, 0-centered for anomalies. Comparable panels share cmap and levels via **one shared `BoundaryNorm`** unless units differ.
- Levels must be scientifically meaningful: no washed-out signals, saturation, false boundaries, or scales that exaggerate noise. Pass `extend="both"` (or `"max"` / `"min"` for one-sided fields) so out-of-range values render as extension triangles, not silent clipping.
- Arial for new figures; preserve an existing project font when patching to avoid style drift. Units in exponential form: `W m−2`, `kg m−2 s−1`, `m s−1` — never slash notation.
- Maps: show latitude/longitude, add coastlines, omit national/provincial borders unless scientifically needed. Pass `transform=ccrs.PlateCarree()` (or the data CRS) to every Cartopy plot call.
- Vectors: scientifically interpretable reference magnitude rounded near the median speed (not the maximum); skip density that reveals circulation without clutter or empty fields; top-right quiver key.
- Significance: visible but subordinate; subsampled dots/stippling; positive and negative signs distinguishable without obscuring the field.
- Bold panel letters in upper-left only; descriptions in upper-right or row/column labels. No `suptitle`, default centered axis titles, captions, or data stamps.
- Prefer low-saturation palettes with visible contrast between adjacent colors; avoid pure primary fills.
- Export PNG at 600 ppi (`dpi=600`, `bbox_inches="tight"`); call `plt.close(fig)` after every figure.

## RR Rules

Full procedure in `references/review.md`. Required:

- Open the rendered PNG every iteration — code review alone is forbidden.
- Per iteration: layout first (colorbar proportion, panel balance, overlap, clipping) → cross-check `plot-standards.md` + Reject Conditions → physics (range, units, sign convention, pressure-axis orientation) → classify `PASS` / `REVISE` / `BLOCKED`.
- Run at least one RR iteration per figure; never declare `PASS` without it. Max 10 iterations; if not `PASS` by then, output `BLOCKED`.
- Top-journal gate — revise if the figure would look out of place in a top atmospheric-science journal: cluttered, unbalanced, or inconsistently styled.

## Patch Mode

For figure-only fixes: read existing code, make the smallest safe change, re-render only affected figures, then RR. Do not rewrite compute code or reorganize directories.

If RR or inspection reveals a compute bug (wrong variable, unit error, bad climatology baseline, missing area weighting, etc.), stop patching and escalate to the Full Workflow from step 2 (Plan).
