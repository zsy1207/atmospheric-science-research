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

## 3. Data validation after opening

After opening any dataset, verify before proceeding:

```python
# 1. Check variables, dimensions, coordinates
print(ds)
# 2. Check units attribute
for var in ds.data_vars:
    print(f"{var}: units={ds[var].attrs.get('units', 'MISSING')}, "
          f"shape={ds[var].shape}, dtype={ds[var].dtype}")
# 3. Check time range
if "time" in ds.dims:
    print(f"Time: {ds.time.values[0]} to {ds.time.values[-1]}, n={ds.time.size}")
# 4. Check spatial extent
for coord in ["lat", "latitude", "lon", "longitude"]:
    if coord in ds.coords:
        print(f"{coord}: {float(ds[coord].min()):.2f} to {float(ds[coord].max()):.2f}")
# 5. Check for missing data
for var in ds.data_vars:
    n_nan = int(ds[var].isnull().sum())
    if n_nan > 0:
        print(f"WARNING: {var} has {n_nan} NaN values ({n_nan/ds[var].size*100:.1f}%)")
```

If units, dimensions, or coordinates are unexpected, stop and clarify before computing.

## 4. Common diagnostic patterns

### 4.1 Climatology and anomaly

```python
# Monthly climatology (reference period)
clim = ds.sel(time=slice("1991-01-01", "2020-12-31")).groupby("time.month").mean("time")

# Monthly anomaly
anom = ds.groupby("time.month") - clim

# Seasonal mean (DJF, MAM, JJA, SON)
seasonal = ds.resample(time="QS-DEC").mean("time")

# Annual mean
annual = ds.resample(time="YS").mean("time")
```

### 4.2 Trend analysis

```python
from scipy.stats import linregress
import numpy as np

def calc_trend(y):
    """Return slope, p-value per grid point."""
    x = np.arange(len(y))
    mask = ~np.isnan(y)
    if mask.sum() < 3:
        return np.nan, np.nan
    res = linregress(x[mask], y[mask])
    return res.slope, res.pvalue

# Apply to xarray with apply_ufunc for vectorization
slope = xr.apply_ufunc(
    lambda y: calc_trend(y)[0],
    ds[var],
    input_core_dims=[["time"]],
    vectorize=True,
    dask="parallelized",
    output_dtypes=[float],
)
```

### 4.3 Composite analysis

```python
# Event-based composite
event_times = [...]  # list of datetime-like values
composite_mean = ds.sel(time=event_times, method="nearest").mean("time")

# Composite difference
pos_composite = ds.sel(time=pos_events).mean("time")
neg_composite = ds.sel(time=neg_events).mean("time")
diff = pos_composite - neg_composite
```

### 4.4 Correlation and regression

```python
from scipy.stats import pearsonr

def corr_with_pvalue(x, y):
    """Pearson correlation with p-value, handling NaN."""
    mask = ~(np.isnan(x) | np.isnan(y))
    if mask.sum() < 3:
        return np.nan, np.nan
    return pearsonr(x[mask], y[mask])
```

### 4.5 EOF / PCA

```python
from eofs.xarray import Eof

# Weight by sqrt(cos(lat)) for area weighting
coslat = np.cos(np.deg2rad(ds.lat))
wgts = np.sqrt(coslat).values
solver = Eof(anom_field, weights=wgts)
eofs = solver.eofs(neofs=3)
pcs = solver.pcs(npcs=3, pcscaling=1)
variance_fractions = solver.varianceFraction(neigs=3)
```

### 4.6 Significance testing

```python
from scipy.stats import ttest_ind, ttest_1samp

# Two-sample t-test (e.g., composite difference significance)
t_stat, p_value = xr.apply_ufunc(
    lambda a, b: ttest_ind(a, b, nan_policy="omit").pvalue,
    group_a, group_b,
    input_core_dims=[["time"], ["time"]],
    vectorize=True,
    dask="parallelized",
    output_dtypes=[float],
)
sig_mask = p_value < 0.05  # for stippling
```

## 5. Work efficiently

- Stay lazy by default; do not call `.load()` early.
- Do not call `.compute()` repeatedly inside loops.
- Do not convert full large arrays directly with `.values` or `.to_numpy()`.
- Subset first, then compute.
- Vectorize first, then parallelize.
- For very large datasets (> 10 GB), explicitly set chunk sizes matching the access pattern (e.g., `chunks={"time": -1, "lat": 100, "lon": 100}` for time-series analysis per spatial chunk).

## 6. Saving intermediate results

```python
# Save processed NetCDF with compression
ds_out.to_netcdf(
    "data_processed/output.nc",
    engine="h5netcdf",
    encoding={var: {"zlib": True, "complevel": 4} for var in ds_out.data_vars},
)
```

- Always include units, long_name, and other essential attributes in output variables.
- Verify the saved file by reopening and printing a summary.

## 7. Optional accelerators

Use these only when the task needs them and they already fit the project stack:
- `metpy` for thermodynamic and atmospheric diagnostics
- `eofs` for EOF or PCA workflows
- `xesmf` for regridding
- `zarr` for large output datasets
- `windspharm` for spherical harmonic wind analysis

Prefer `scipy` for interpolation, filtering, statistical tests, regression, and correlation.

## 8. Prohibited patterns

- Do not run heavy computation inside plotting scripts.
- Do not rerun the full diagnostic chain just to change colors, titles, labels, or layout.
- Do not write processed outputs back into the raw-data directory.
- Do not load entire datasets into memory when only a spatial or temporal subset is needed.
- Do not use nested Python loops over grid points when vectorized or `apply_ufunc` alternatives exist.
