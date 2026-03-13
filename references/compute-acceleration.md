# Compute Acceleration

Use this file for preprocessing, diagnostics, statistics, regridding, or expensive workloads.

## 1. Base rules

- Keep compute scripts separate from plot scripts.
- Compute shared diagnostics once, save to `data_processed/` (or project equivalent), reuse.
- For figure-only patches, do not add new compute steps unless the diagnostic is wrong.
- Prioritize dask, h5netcdf, and vectorized algorithms.

## 2. Dataset opening

```python
# Single NetCDF
xr.open_dataset(path, engine="h5netcdf", chunks="auto")

# Multiple NetCDF
xr.open_mfdataset(paths, combine="by_coords", parallel=True, engine="h5netcdf", chunks="auto")

# GRIB
xr.open_dataset(path, engine="cfgrib", chunks="auto")
```

## 3. Data validation

After opening, verify: variables, dimensions, coordinates, units, time range, spatial extent, NaN count. Stop and clarify if unexpected.

## 4. Common patterns

```python
# Climatology & anomaly
clim = ds.sel(time=slice("1991", "2020")).groupby("time.month").mean("time")
anom = ds.groupby("time.month") - clim

# Seasonal / annual mean
seasonal = ds.resample(time="QS-DEC").mean("time")
annual = ds.resample(time="YS").mean("time")

# Trend (vectorized)
from scipy.stats import linregress
def calc_trend(y):
    x = np.arange(len(y))
    mask = ~np.isnan(y)
    if mask.sum() < 3: return np.nan, np.nan
    res = linregress(x[mask], y[mask])
    return res.slope, res.pvalue

slope = xr.apply_ufunc(lambda y: calc_trend(y)[0], ds[var],
    input_core_dims=[["time"]], vectorize=True, dask="parallelized", output_dtypes=[float])

# Composite
composite = ds.sel(time=event_times, method="nearest").mean("time")

# Correlation
from scipy.stats import pearsonr
# use apply_ufunc to vectorize pearsonr across grid

# EOF / PCA
from eofs.xarray import Eof
wgts = np.sqrt(np.cos(np.deg2rad(ds.lat))).values  # area weighting
solver = Eof(anom_field, weights=wgts)
eofs = solver.eofs(neofs=3); pcs = solver.pcs(npcs=3, pcscaling=1)

# Significance (t-test)
from scipy.stats import ttest_ind
p_value = xr.apply_ufunc(lambda a, b: ttest_ind(a, b, nan_policy="omit").pvalue,
    group_a, group_b, input_core_dims=[["time"], ["time"]],
    vectorize=True, dask="parallelized", output_dtypes=[float])
sig_mask = p_value < 0.05
```

## 5. Efficiency rules

- Stay lazy; do not `.load()` or `.compute()` early or inside loops.
- Subset first, then compute. Vectorize first, then parallelize.
- For > 10 GB datasets, set chunk sizes matching the access pattern.
- Do not use nested Python loops over grid points when `apply_ufunc` works.

## 6. Saving

```python
ds_out.to_netcdf("data_processed/output.nc", engine="h5netcdf",
    encoding={v: {"zlib": True, "complevel": 4} for v in ds_out.data_vars})
```

Include units and long_name in output attributes. Verify by reopening.

## 7. Optional packages

Use only when needed: `metpy`, `eofs`, `xesmf`, `zarr`, `windspharm`. Prefer `scipy` for stats and interpolation.
