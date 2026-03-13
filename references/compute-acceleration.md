# Compute Acceleration

Use this file for preprocessing, diagnostics, statistics, regridding, or any workload that can become expensive.

## 1. Base rules

- Prioritize using faster packages such as dask and h5netcdf, along with more efficient algorithms.
- Keep compute scripts separate from plot scripts.
- Keep diagnostics and statistics scientifically defensible.
- Compute shared diagnostics once, save them in the project's established processed-data location or `data_processed/`, and reuse them.
- For figure-only patches, do not add new compute steps unless the existing diagnostic is actually wrong.

## 2. Dataset opening defaults

### 2.1 Single NetCDF

```python
xr.open_dataset(path, engine="h5netcdf", chunks="auto")
```

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

### 2.3 GRIB

```python
xr.open_dataset(path, engine="cfgrib", chunks="auto")
```

## 3. Work efficiently

- Stay lazy by default; do not call `.load()` early.
- Do not call `.compute()` repeatedly inside loops.
- Do not convert full large arrays directly with `.values` or `.to_numpy()`.
- Subset first, then compute.
- Vectorize first, then parallelize.


## 4. Optional accelerators

Use these only when the task needs them and they already fit the project stack:
- `metpy` for thermodynamic and atmospheric diagnostics
- `eofs` for EOF or PCA workflows
- `xesmf` for regridding
- `zarr` for outputs data
Prefer `scipy` for interpolation, filtering, statistical tests, regression, and correlation.


## 5. Prohibited patterns

- Do not run heavy computation inside plotting scripts.
- Do not rerun the full diagnostic chain just to change colors, titles, labels, or layout.
- Do not write processed outputs back into the raw-data directory.
