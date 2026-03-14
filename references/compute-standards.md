# Compute Patterns

## Coordinate Normalization — do FIRST, before any merge or comparison

Different datasets use different names and conventions. Normalizing upfront prevents silent spatial misalignment.

```python
# Rename inconsistent coordinate names
rename = {}
for old, new in [("latitude", "lat"), ("longitude", "lon"), ("level", "lev")]:
    if old in ds.coords or old in ds.dims:
        rename[old] = new
if rename:
    ds = ds.rename(rename)

# Longitude: 0–360 → −180/180 (standard for regional studies)
ds = ds.assign_coords(lon=(ds.lon + 180) % 360 - 180).sortby("lon")

# Longitude: −180/180 → 0–360 (standard for Pacific-centered studies)
# ds = ds.assign_coords(lon=ds.lon % 360).sortby("lon")

# Ensure lat south-to-north (required for most regridding and comparison)
if ds.lat[0] > ds.lat[-1]:
    ds = ds.sortby("lat")
```

## Dataset I/O

```python
# NetCDF — always use h5netcdf for speed + lazy loading
xr.open_dataset(path, engine="h5netcdf", chunks="auto")
xr.open_mfdataset(paths, combine="by_coords", parallel=True, engine="h5netcdf", chunks="auto")

# GRIB — cfgrib is the only reliable engine
xr.open_dataset(path, engine="cfgrib", chunks="auto")
```

### Saving

Include `units` and `long_name` attributes so downstream code can self-document. Verify by reopening.

```python
ds_out.to_netcdf("data_processed/output.nc", engine="h5netcdf",
    encoding={v: {"zlib": True, "complevel": 4} for v in ds_out.data_vars})
```

## Data Validation — call after every open or save

```python
def validate_dataset(ds, expected_vars=None):
    """Print summary of dataset dimensions, ranges, and NaN coverage."""
    print("Dims:", dict(ds.dims))
    print("Coords:", list(ds.coords))
    for v in (expected_vars or ds.data_vars):
        d = ds[v]
        print(f"  {v}: shape={d.shape}, dtype={d.dtype}, "
              f"range=[{float(d.min()):.4g}, {float(d.max()):.4g}], "
              f"NaN%={float(d.isnull().mean())*100:.1f}, "
              f"units={d.attrs.get('units', 'MISSING')}")
    if "time" in ds.coords:
        print(f"Time: {ds.time.values[0]} → {ds.time.values[-1]} ({ds.dims['time']} steps)")
```

## Climatology & Anomaly

These are the most common operations in atmospheric diagnostics. Always specify the climatology baseline period explicitly.

```python
# Monthly climatology (long-term monthly mean)
clim = ds[var].sel(time=slice("1991", "2020")).groupby("time.month").mean("time")

# Monthly anomaly relative to climatology
anom = ds[var].groupby("time.month") - clim

# Seasonal mean (DJF, MAM, JJA, SON)
# resample with QS-DEC so DJF groups Dec of previous year with Jan-Feb
seasonal = ds[var].resample(time="QS-DEC").mean()

# Annual mean
annual = ds[var].resample(time="YS").mean()
```

## Composite Analysis

```python
# Select specific dates/events and compute composite mean
composite = ds[var].sel(time=event_dates, method="nearest").mean("time")

# Composite anomaly (subtract climatology first)
composite_anom = anom.sel(time=event_dates, method="nearest").mean("time")
```

## Trend & Detrending

```python
# Linear detrending (vectorized, works with dask)
from scipy.signal import detrend
detrended = xr.apply_ufunc(detrend, ds[var],
    input_core_dims=[["time"]], output_core_dims=[["time"]],
    vectorize=True, dask="parallelized", output_dtypes=[float])

# Linear trend (slope per year) at each grid point
def _linear_slope(y):
    """Return slope of least-squares fit per time step."""
    import numpy as np
    x = np.arange(len(y), dtype=float)
    mask = ~np.isnan(y)
    if mask.sum() < 3:
        return np.nan
    return np.polyfit(x[mask], y[mask], 1)[0]

slope = xr.apply_ufunc(_linear_slope, ds[var],
    input_core_dims=[["time"]], vectorize=True,
    dask="parallelized", output_dtypes=[float])
```

## Spatial & Statistical Patterns

```python
# Area-weighted spatial mean (cosine latitude weighting)
weights = np.cos(np.deg2rad(ds.lat))
ds_mean = ds[var].weighted(weights).mean(dim=["lat", "lon"])

# EOF analysis with area weighting (sqrt-cosine for EOF)
from eofs.xarray import Eof
wgts = np.sqrt(np.cos(np.deg2rad(ds.lat))).values
solver = Eof(anom_field, weights=wgts)

# Vectorized grid-point computation (apply_ufunc, never nested Python loops)
result = xr.apply_ufunc(calc_func, ds[var],
    input_core_dims=[["time"]], vectorize=True,
    dask="parallelized", output_dtypes=[float])

# Regridding (xesmf)
import xesmf as xe
regridder = xe.Regridder(ds, ds_out, "bilinear", periodic=True)
# Build regridder once, reuse for all variables/time steps
```

## Significance Testing

```python
# Two-sample t-test at each grid point (e.g., composite vs climatology)
from scipy import stats

def _ttest(a, b):
    """Two-sample t-test, return p-value."""
    t, p = stats.ttest_ind(a, b, nan_policy="omit")
    return p

p_value = xr.apply_ufunc(_ttest, group_a, group_b,
    input_core_dims=[["time"], ["time"]], vectorize=True,
    dask="parallelized", output_dtypes=[float])
```

## Acceleration & Efficiency

- **Chunking**: set `chunks="auto"` at open time. Do not rechunk afterwards — let dask pick the layout once.
- **Subset first**: select region/time/level before heavy computation. Drop unused variables with `ds = ds[needed_vars]` early.
- **Vectorize**: use `apply_ufunc` with `dask="parallelized"`, never nested Python loops over grid points.
- **Faster groupby**: `flox` dramatically speeds up `groupby().mean()` — use `ds.groupby("time.month").mean(method="cohorts")` if available.
- **Avoid repeated I/O**: open datasets once and pass the xarray object; do not re-read files inside loops.
- **Regridding**: build the `xesmf.Regridder` once, reuse for all variables/time steps. Use `periodic=True` for global lon.
- **PyTorch**: when using `torch`, use Intel XPU as the device (`torch.xpu`). `tensor = torch.tensor([1.0, 2.0]).to("xpu")`

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---|---|---|
| Lon convention mismatch | Map shows data shifted 180° or has white gap at dateline | Normalize lon convention before plotting or merging |
| Lat reversed | Spatial patterns appear upside-down | `ds.sortby("lat")` to ensure south→north |
| Missing `units` attribute | Downstream code cannot auto-label axes or convert units | Always set `ds[var].attrs["units"]` before saving |
| Integer overflow in time | Dates show as nanoseconds or wrap around | Use `decode_times=True` (default) or convert with `pd.to_datetime` |
| Mixing chunked/unchunked | `compute()` hangs or OOM on large data | Either chunk everything or nothing; don't mix |
| Silent NaN propagation | Mean/sum returns NaN for entire region | Use `skipna=True` or check for all-NaN slices |
| Regridder weight file conflict | `xesmf` throws error on second run | Set `reuse_weights=True` or delete old weight files |
