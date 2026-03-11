# Compute Acceleration

Use this file for preprocessing, diagnostics, statistics, regridding, or any workload that can become expensive.
After reading it, pick the lightest stack that fits the task and keep compute separate from plotting.

## 1. Base rules

- Raw inputs are read from `~/code/data` by default unless the task says otherwise. Do not write processed outputs back into `~/code/data`.
- Prioritize using faster packages such as dask and h5netcdf, along with more efficient algorithms.
- Before choosing packages, inspect the current environment and the project's established stack. Keep the fastest supported path that is already available.
- Keep compute scripts separate from plot scripts.
- Keep diagnostics and statistics scientifically defensible.
- Compute shared diagnostics once, save them in the project's established processed-data location or `data_processed/`, and reuse them.
- Prefer the fastest suitable approach already available in the project environment; do not add package churn without a reason.
- Prefer direct compute paths over one-off wrappers or extra configurability that this task does not need.
- For figure-only patches, do not add new compute steps unless the existing diagnostic is actually wrong.

## 2. Dataset opening defaults

### 2.1 Single NetCDF

```python
xr.open_dataset(path, engine="h5netcdf", chunks="auto")
```

If `h5netcdf` is unavailable, fails, or reads incorrectly, fall back to `netcdf4`. If the existing project relies on another stable `xarray` engine, reuse that instead of switching stacks.

### 2.2 Multiple NetCDF files

```python
xr.open_mfdataset(
    paths,
    combine="by_coords",
    parallel=True,
    engine="h5netcdf",
    chunks="auto",
)
```

If `h5netcdf` is unavailable or unstable for the dataset, fall back to `netcdf4` or the existing project default and keep the same lazy-loading strategy.

### 2.3 GRIB

```python
xr.open_dataset(path, engine="cfgrib", chunks="auto")
```

Use `cfgrib` when it is available. If it is unavailable, use an existing project conversion path or another lightweight metadata probe already present in the environment, and record the fallback instead of installing new packages mid-task.

## 3. Work efficiently

- Stay lazy by default; do not call `.load()` early.
- Do not call `.compute()` repeatedly inside loops.
- Do not convert full large arrays directly with `.values` or `.to_numpy()`.
- Subset first, then compute.
- Vectorize first, then parallelize.
- Start from `chunks="auto"` and avoid repeated `rechunk`.
- If the main task is temporal aggregation or statistics, keep chunks friendly to `time`.
- If the main task is spatial regridding or large-area map processing, avoid overly large spatial chunks that cause memory spikes.

Prefer native `xarray` operations such as `groupby`, `resample`, `coarsen`, `rolling`, and `weighted`.

## 4. Optional accelerators

Use these only when the task needs them and the environment already supports them:
- `flox` for grouped reductions on chunked data
- `bottleneck` for fast `nan`-heavy reductions
- `metpy` for thermodynamic and atmospheric diagnostics
- `eofs` for EOF or PCA workflows
- `xesmf` for regridding

Prefer `scipy` for interpolation, filtering, statistical tests, regression, and correlation.

If the project already has an equivalent, established stack, reuse it instead of switching libraries.

## 5. Checkpoints and outputs

- Only call `.compute()` or write to disk at necessary checkpoints.
- If an expensive intermediate will be reused, `persist()` it or write it to disk once it is ready.
- Prefer `zarr` for large outputs that will be read repeatedly, when the environment already supports it cleanly.
- Use `NetCDF` for lighter or final exchange outputs.
- Save shared diagnostics in the project's established processed-data location. If no such location exists, use `data_processed/<name>.nc` or `data_processed/<name>.zarr`.
- Plotting code, whether it lives in `figN/plot_*.py` or the project's existing figure scripts, should read prepared outputs instead of rebuilding the full diagnostic chain.
- Separate compute scripts are optional; use them only when direct reads from the processed outputs are not enough.
- When a diagnostic changes, rerun only the affected compute chain and downstream figures, not the whole project.


## 6. Minimum acceptance for non-figure outputs

- Confirm the output file exists at the intended path before treating the compute stage as done.
- Inspect the saved output summary: key variable names, dimensions, coordinates, and units.
- Check missing-data behavior with a quick `NaN` or mask summary.
- Check value range, sign, and order of magnitude with min/max or representative quantiles.
- Confirm time coverage, region, and grid shape match the intended diagnostic.
- If any of these fail, revise the affected compute stage before plotting.

## 7. Prohibited patterns

- Do not run heavy computation inside plotting scripts.
- Do not rerun the full diagnostic chain just to change colors, titles, labels, or layout.
- Do not write processed outputs into the raw-data directory.
