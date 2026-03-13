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

After opening, run through this checklist. Stop and clarify with the user if anything is unexpected.

```python
def validate_dataset(ds, expected_vars=None):
    """Quick validation — call immediately after opening."""
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
    if "lat" in ds.coords:
        print(f"Lat: {float(ds.lat.min()):.2f} → {float(ds.lat.max()):.2f}")
    if "lon" in ds.coords:
        print(f"Lon: {float(ds.lon.min()):.2f} → {float(ds.lon.max()):.2f}")
```

Key checks:
- Variables present and correctly named
- Dimensions in expected order (time, level, lat, lon)
- Units attribute present; physically correct
- Value range plausible (no fill values leaking as real data, e.g., −9999)
- NaN fraction reasonable; no all-NaN time steps or spatial slices
- Time decoded as datetime (not raw float/int)
- Spatial extent covers the target region

## 3.5. Coordinate normalization

Handle before any merge, comparison, or plotting across datasets.

```python
# Longitude: 0–360 → −180–180
ds = ds.assign_coords(lon=(ds.lon + 180) % 360 - 180).sortby("lon")

# Longitude: −180–180 → 0–360
ds = ds.assign_coords(lon=ds.lon % 360).sortby("lon")

# Rename inconsistent coordinate names
rename_map = {}
for old, new in [("latitude", "lat"), ("longitude", "lon"), ("level", "lev")]:
    if old in ds.coords or old in ds.dims:
        rename_map[old] = new
if rename_map:
    ds = ds.rename(rename_map)

# Ensure lat is south-to-north
if ds.lat[0] > ds.lat[-1]:
    ds = ds.sortby("lat")

# Ensure pressure levels are top-to-bottom (100→1000) or bottom-to-top depending on need
# if ds.lev[0] > ds.lev[-1]:
#     ds = ds.sortby("lev")
```

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

# Detrend (vectorized)
from scipy.signal import detrend as _detrend
detrended = xr.apply_ufunc(_detrend, ds[var],
    input_core_dims=[["time"]], output_core_dims=[["time"]],
    vectorize=True, dask="parallelized", output_dtypes=[float],
    kwargs={"type": "linear", "axis": -1})

# Running mean / rolling
rolling_mean = ds[var].rolling(time=12, center=True, min_periods=6).mean()

# Bandpass filter (Lanczos weights)
from scipy.signal import firwin, filtfilt
def bandpass(y, fs, low, high, numtaps=121):
    nyq = fs / 2
    b = firwin(numtaps, [low/nyq, high/nyq], pass_zero=False)
    mask = ~np.isnan(y)
    if mask.sum() < numtaps: return np.full_like(y, np.nan)
    out = np.full_like(y, np.nan)
    out[mask] = filtfilt(b, 1, y[mask])
    return out

# Regridding (xesmf)
import xesmf as xe
ds_out = xr.Dataset({"lat": (["lat"], np.arange(-90, 91, 1.0)),
                      "lon": (["lon"], np.arange(0, 360, 1.0))})
regridder = xe.Regridder(ds, ds_out, "bilinear", periodic=True)
ds_regridded = regridder(ds[var])

# Area-weighted spatial mean
weights = np.cos(np.deg2rad(ds.lat))
ds_weighted = ds[var].weighted(weights).mean(dim=["lat", "lon"])

# Nino 3.4 index (example region extraction)
nino34 = ds["sst"].sel(lat=slice(-5, 5), lon=slice(190, 240)).mean(dim=["lat", "lon"])
```

## 5. Efficiency rules

- Stay lazy; do not `.load()` or `.compute()` early or inside loops.
- Subset first, then compute. Vectorize first, then parallelize.
- For > 10 GB datasets, set chunk sizes matching the access pattern.
- Chunk sizing guide: time-series analysis → chunk along space; spatial maps → chunk along time. Typical good sizes: `{"time": -1, "lat": 90, "lon": 90}` for time-heavy, `{"time": 120, "lat": -1, "lon": -1}` for space-heavy.
- Do not use nested Python loops over grid points when `apply_ufunc` works.
- For large `apply_ufunc` calls, prefer `dask="parallelized"` over `dask="allowed"` for predictable chunking.
- If a compute step is killed or hangs, reduce chunk size or spatial/temporal subset before retrying.

## 6. Saving

```python
ds_out.to_netcdf("data_processed/output.nc", engine="h5netcdf",
    encoding={v: {"zlib": True, "complevel": 4} for v in ds_out.data_vars})
```

Include units and long_name in output attributes. Verify by reopening.

## 7. Optional packages

Use only when needed: `metpy`, `eofs`, `xesmf`, `zarr`, `windspharm`. Prefer `scipy` for stats and interpolation. Do not install packages mid-task — if a package is missing, use a lightweight alternative from existing imports or surface the dependency to the user.
