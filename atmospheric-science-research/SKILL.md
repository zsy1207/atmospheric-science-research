---
name: atmospheric-science-research
description: Computation and publication-grade plotting for atmospheric and climate research on local data files (NetCDF, GRIB, HDF5, Zarr; ERA5, CMIP, reanalysis). Use whenever the user works with gridded lat/lon or pressure-level data and needs diagnostics (climatology, anomalies, composites, regression/correlation, trends, EOF, extremes) or figures (Cartopy maps, cross-sections, time series), even if they do not say "paper figure". Triggers on 气候态、距平、合成分析、再分析资料、诊断计算、论文配图.
---

# Atmospheric Science Research

Compute diagnostics on local atmospheric data and plot them to journal standard, inside a reproducible project.

## References

| File | Read |
|---|---|
| `references/plot-standards.md` | Before writing or changing plot code |
| `references/review.md` | Before the first review-revision (RR) iteration |
| `references/readme-template.md` | Before writing or updating the README |

## Workflow

New analyses run steps 1–7. Figure-only fixes use Patch Mode.

1. **Probe** — Inspect metadata only: `ncdump -h`, `cdo sinfon`, `h5dump -H`, or a metadata-only Python one-liner for Zarr.
2. **Plan** — Ask only scientific questions that change the result, each with a recommended answer. Record every adopted assumption in the plan and the README.
3. **Build** — `figN/compute_*.py`, `figN/plot_*.py`, `data_output/figN/...`. Do not reorganize existing projects unless asked.
4. **Compute** — Scripts run without required CLI args, are idempotent, and write reusable outputs before plots read them.
5. **Plot** — Follow `plot-standards.md`.
6. **Review** — Run RR per `review.md` until each figure is `PASS` or `BLOCKED`. Open the rendered PNG in every iteration; reviewing code alone never counts.
7. **Export and document** — Export final files (`plot-standards.md` § Export), then write or update one Chinese `README.md`. `BLOCKED` figures and unresolved risks go in its risk section.

## Compute Rules

- **Vectorize** — `xr.where` and boolean masks instead of loops over grid points; `.mean(dim=..., skipna=True)` keeps coords; `xr.corr`, `xr.cov`, and `xr.apply_ufunc(..., dask="parallelized")` for statistics.
- **Area weighting** — Spatial means use cos(lat) weights: `da.weighted(np.cos(np.deg2rad(da.lat))).mean(("lat", "lon"))`.
- **Daily climatology** — `groupby("time.dayofyear")` shifts every date after Feb 28 by one day in leap years, and dropping Feb 29 alone does not fix it. Group by `da.time.dt.strftime("%m-%d")`, or convert a copy with `da.convert_calendar("noleap")` (its time axis becomes cftime, so keep the original for everything else). Then smooth with a 31-day running mean (wraparound) unless the user specifies otherwise.
- **Regridding** — `cdo remapcon` / xesmf `"conservative"` for precipitation and fluxes; `remapbil` / `"bilinear"` for smooth state variables (T, Z, u, v, q).
- **Reading** — `xr.open_dataset(path, engine="h5netcdf", chunks="auto")` or `xr.open_mfdataset(paths, engine="h5netcdf", parallel=True, chunks="auto")`; fall back to the default engine if `h5netcdf` fails.
- **Dask boundary** — Keep raw daily fields lazy, reduce to the coarsest stack the science allows, compute once there, then loop only over in-memory slices. Rechunk `{"time": -1}` for percentiles, detrending, or `apply_ufunc` over `time`. Never `.compute()` inside loops.
- **Regional padding** — Pad subsets by at least 1 grid point beyond the target bounds so maps have no blank border.

## Patch Mode

Read the existing code, make the smallest safe change, re-render only the affected figures, run RR, export, and append a「版本更新」entry to the README.

If RR reveals a compute bug (wrong variable, unit error, bad climatology baseline), stop patching and restart from step 2.
