# Review & Revision (RR)

Bounded, figure-focused loop. Catches visual defects and obvious data-sanity issues; not a full science re-audit.

## Loop

1. Render the figure if needed.
2. Open the actual PNG. Never review code alone.
3. Check **layout** first: subplot balance, colorbar proportion, overlaps, clipping, whitespace, legend placement.
4. Cross-check against `plot-standards.md` and the Reject Conditions table below.
5. Check **physics**: value range, units, sign convention, pressure-axis orientation, spatial pattern.
6. Classify:
   - `PASS` — no visual or obvious data-sanity issue remains.
   - `REVISE` — fix the first material issue, re-render, reopen PNG.
   - `BLOCKED` — user decision, missing input/dependency, or unreadable output.

Run at least one RR iteration per figure; never declare `PASS` without it. Max 10 iterations per figure.

## Top-Journal Visual Gate

Even when no specific rule fails, revise if the figure would look out of place in a top atmospheric-science journal — cluttered, unbalanced, visually noisy, inconsistently styled, or hard to read.

## Reject Conditions

Revise immediately if any item is true.

| Area | Reject if |
|---|---|
| Text | Font not Arial; unstyled default axis title; figure-level `suptitle()`; figure-level caption; source/data stamp; missing units/labels; units not exponential; clipped text; font < 7 pt |
| Panels | Multi-panel without bold panel letters (`a`, `b`, ...) slightly outside the upper-left; descriptions not centered over each panel; quiver keys not top-right aligned to the description baseline; inconsistent panel styling |
| Colormap | Uses `jet` / `rainbow` / `nipy_spectral`; no explicit `levels`; missing `extend=` when values may exceed `levels`; anomaly not 0-centered; comparable panels use independent norms instead of one shared `BoundaryNorm` |
| Geography | Wrong projection; missing `transform=ccrs.PlateCarree()` on `contourf` / `pcolormesh` / `quiver` / `scatter`; missing coastlines; unnecessary national/provincial borders; missing latitude/longitude indication; dateline seam; missing relevant inset |
| Tibet | ≤ 850 hPa field over Tibet not masked |
| Vectors | Too dense / sparse; missing quiver key; bad key position or magnitude |
| Significance | Invisible; dot size inappropriate; not subsampled when needed; obscures signal |
| Layout | Crowded panels; dominant colorbar; overlap; clipping; poor whitespace |
| Physics | Implausible values; wrong sign convention; wrong pressure-axis orientation; missing or wrong units |

## Quick Fixes

Starting points; tune against the rendered PNG.

| Defect | First fix |
|---|---|
| Colorbar overlaps or dominates | `shrink=0.5–0.8`, `pad=0.06–0.10`, `aspect=25–40`, or dedicated `GridSpec` slot |
| Panel label clips | Place the letter slightly outside the upper-left with `ax.text(..., transform=ax.transAxes, clip_on=False)`; increase top/left margin |
| Quiver key overlaps description | Move to top-right on the description baseline: `quiverkey(Q, 1.0, 1.04, ..., labelpos="W")` |
| Quiver too dense / sparse | Adjust skip: 2.5° → 1–2, 1° → 3–5, 0.25° → 8–15 |
| Ticks overlap | Fewer ticks via `FixedLocator` / `MaxNLocator`; reduce font only if still readable |
| Gridline edge labels collide | `gl.top_labels = False; gl.right_labels = False` |
| Dateline seam | Check `transform=ccrs.PlateCarree()`, cyclic point, central longitude |
| Map warped, empty, or off-axis | Add `transform=ccrs.PlateCarree()` to every Cartopy plot call |
| Anomaly not centered | Symmetric levels around 0 + diverging `cmaps` |
| Out-of-range fill flat at min/max | `extend="both"` on `contourf` + colorbar (`"max"` / `"min"` if one-sided, e.g. precip) |
| Fill washed out / saturated | Inspect summary range; reset common interpretable levels |
| Stippling invisible | Increase marker size / alpha or use sparse hatching `"..."` |
| Stippling too dense | Increase subsampling or reduce marker size / alpha |
| Legend hides data | Move outside axes with `bbox_to_anchor` |
| Cross-section / profile wrong | Invert pressure axis; verify hPa units and ordering |
| Tibet not masked | Add grey Tibet polygon; hide low-level arrows over Tibet |
| Layout still wrong | Change figure geometry or `GridSpec`; code parameters are only estimates |
