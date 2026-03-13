# Compute Patterns

## Coordinate normalization — do FIRST, before any merge or comparison

```python
# Rename inconsistent coordinate names
rename = {}
for old, new in [("latitude", "lat"), ("longitude", "lon"), ("level", "lev")]:
    if old in ds.coords or old in ds.dims:
        rename[old] = new
if rename:
    ds = ds.rename(rename)

# Longitude: 0–360 → −180/180
ds = ds.assign_coords(lon=(ds.lon + 180) % 360 - 180).sortby("lon")

# Longitude: −180/180 → 0–360
# ds = ds.assign_coords(lon=ds.lon % 360).sortby("lon")

# Ensure lat south-to-north
if ds.lat[0] > ds.lat[-1]:
    ds = ds.sortby("lat")
```

## Data validation — call after every open or save

```python
def validate_dataset(ds, expected_vars=None):
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

## Dataset opening

```python
xr.open_dataset(path, engine="h5netcdf", chunks="auto")
xr.open_mfdataset(paths, combine="by_coords", parallel=True, engine="h5netcdf", chunks="auto")
xr.open_dataset(path, engine="cfgrib", chunks="auto")  # GRIB
```

## Key patterns

```python
# Area-weighted spatial mean (cos weighting)
weights = np.cos(np.deg2rad(ds.lat))
ds_mean = ds[var].weighted(weights).mean(dim=["lat", "lon"])

# EOF with area weighting (sqrt cos for EOF)
from eofs.xarray import Eof
wgts = np.sqrt(np.cos(np.deg2rad(ds.lat))).values
solver = Eof(anom_field, weights=wgts)

# Vectorized trend (apply_ufunc, not Python loops)
slope = xr.apply_ufunc(calc_func, ds[var],
    input_core_dims=[["time"]], vectorize=True,
    dask="parallelized", output_dtypes=[float])

# Regridding (xesmf)
import xesmf as xe
regridder = xe.Regridder(ds, ds_out, "bilinear", periodic=True)
```

## Saving

```python
ds_out.to_netcdf("data_processed/output.nc", engine="h5netcdf",
    encoding={v: {"zlib": True, "complevel": 4} for v in ds_out.data_vars})
```

MUST include `units` and `long_name` in attributes. Verify by reopening.

## Acceleration & efficiency

- **Chunking**: set `chunks="auto"` at open time. NEVER rechunk afterwards — let dask pick the layout once.
- **Subset first**: select region/time/level before heavy computation. Drop unused variables with `ds = ds[needed_vars]` early.
- **Vectorize**: use `apply_ufunc` with `dask="parallelized"`, NEVER nested Python loops over grid points.
- **Faster groupby**: `flox` dramatically speeds up `groupby().mean()` — use `ds.groupby("time.month").mean(method="cohorts")` if available.
- **Avoid repeated I/O**: open datasets once and pass the xarray object; do not re-read files inside loops.
- **Regridding**: build the `xesmf.Regridder` once, reuse for all variables/time steps. Use `periodic=True` for global lon.
- **PyTorch**: when using `torch`, MUST use Intel XPU as the device (`torch.xpu`). `tensor = torch.tensor([1.0, 2.0]).to(“xpu”)`
- Use only existing packages. If a package is missing, use a lightweight alternative or surface the dependency.
