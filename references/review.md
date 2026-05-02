# Review & Revision (RR)

Bounded, figure-focused loop. Catches visual defects and obvious data-sanity problems; not a full science re-audit.

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

Run at least one RR iteration per figure — never declare `PASS` without going through the loop. Max 10 iterations per figure.

## Top-Journal Visual Gate

Even when no specific rule fails, revise if the figure would look out of place in a top atmospheric-science journal — cluttered, unbalanced, visually noisy, inconsistently styled, or hard to read.

## Reject Conditions

Revise immediately if any item is true.

| Area | Reject if |
|---|---|
| Text | Centered title, `suptitle()`, source/data stamp, missing units/labels, clipped text, font < 7 pt |
| Panels | Multi-panel without `(a)` labels; inconsistent panel styling |
| Colormap | Not `cmaps`; no explicit levels; anomaly not centered on 0; incomparable panel scales |
| Geography | Wrong projection, missing coastlines, dateline seam, missing relevant border / inset |
| Tibet | ≤ 850 hPa field over Tibet not masked |
| Vectors | Too dense / sparse, missing quiver key, bad key position or magnitude |
| Significance | Invisible or obscures signal |
| Layout | Crowded panels, dominant colorbar, overlap, clipping, poor whitespace |
| Physics | Implausible values, wrong sign convention, wrong pressure-axis orientation, missing or wrong units |

## Quick Fixes

Starting points; tune against the rendered PNG.

| Defect | First fix |
|---|---|
| Colorbar overlaps or dominates | `shrink=0.5–0.8`, `pad=0.06–0.10`, `aspect=25–40`, or dedicated `GridSpec` slot |
| Panel label clips | `ax.set_title(..., loc="left")`; increase top margin |
| Quiver key overlaps label | Move to top-right: `quiverkey(Q, 1.0, 1.04, ..., labelpos="W")` |
| Quiver too dense / sparse | Adjust skip: 2.5° → 1–2, 1° → 3–5, 0.25° → 8–15 |
| Ticks overlap | Fewer ticks via `FixedLocator` / `MaxNLocator`; reduce font only if still readable |
| Gridline edge labels collide | `gl.top_labels = False; gl.right_labels = False` |
| Dateline seam | Check `transform=ccrs.PlateCarree()`, cyclic point, central longitude |
| Anomaly not centered | Symmetric levels around 0 + diverging `cmaps` |
| Fill washed out / saturated | Inspect summary range; reset common interpretable levels |
| Stippling invisible | Increase marker size / alpha or use sparse hatching `"..."` |
| Stippling too dense | Increase subsampling or reduce marker size / alpha |
| Legend hides data | Move outside axes with `bbox_to_anchor` |
| Cross-section / profile wrong | Invert pressure axis; verify hPa units and ordering |
| Tibet not masked | Add grey Tibet polygon; hide low-level arrows over Tibet |
| Layout still wrong | Change figure geometry or `GridSpec`; code parameters are only estimates |
