# Compute Standards

## Coordinate Normalization — do FIRST

```python
rename = {}
for old, new in [("latitude", "lat"), ("longitude", "lon"), ("level", "lev")]:
    if old in ds.coords or old in ds.dims:
        rename[old] = new
if rename:
    ds = ds.rename(rename)

# Longitude: 0–360 → −180/180
ds = ds.assign_coords(lon=(ds.lon + 180) % 360 - 180).sortby("lon")
# Reverse: ds = ds.assign_coords(lon=ds.lon % 360).sortby("lon")

# Ensure lat south-to-north
if ds.lat[0] > ds.lat[-1]:
    ds = ds.sortby("lat")
```

## Dataset I/O

```python
xr.open_dataset(path, engine="h5netcdf", chunks="auto")
xr.open_mfdataset(paths, combine="by_coords", parallel=True, engine="h5netcdf", chunks="auto")
xr.open_dataset(path, engine="cfgrib", chunks="auto")  # GRIB only

# Save with compression — always include units and long_name attrs
ds.to_netcdf(path, engine="h5netcdf",
    encoding={v: {"zlib": True, "complevel": 4} for v in ds.data_vars})
```

## Performance Rules

1. **Subset first** — select region, time, level and `ds = ds[needed_vars]` before any heavy operation.
2. **Never loop over grid points** — use `xr.apply_ufunc(..., vectorize=True, dask="parallelized")` or vectorized numpy/scipy.
3. **Chunk once at open time** — `chunks="auto"` on open; do not rechunk afterwards.
4. **Faster groupby** — `ds.groupby("time.month").mean(method="cohorts")` via flox.
5. **Open once, pass objects** — never re-read files inside loops.
6. **Build regridder once** — `xesmf.Regridder` with `periodic=True`; reuse for all fields.
7. **Stay lazy** — delay `.compute()` / `.values` until the final step.
8. **Area-weight with cosine** — `np.cos(np.deg2rad(ds.lat))` for mean; `np.sqrt(...)` for EOF.
9. **PyTorch on XPU** — `tensor.to("xpu")` for Intel XPU acceleration.

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---|---|---|
| Lon mismatch | Map shifted 180° / white gap | Normalize lon first |
| Lat reversed | Patterns upside-down | `ds.sortby("lat")` |
| Missing units | Auto-label fails | Set `attrs["units"]` before save |
| Time overflow | Dates as nanoseconds | `decode_times=True` or `pd.to_datetime` |
| Chunked/unchunked mix | OOM or hang | Chunk everything or nothing |
| NaN propagation | Region returns NaN | `skipna=True` |
| Regridder conflict | xesmf error on second run | `reuse_weights=True` |
