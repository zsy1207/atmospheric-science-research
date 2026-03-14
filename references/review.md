# Review & Revision (RR)

RR is a **figure-focused** review loop. The goal is to catch and fix visual defects and data sanity issues — not to re-audit the science.

## Steps

1. **Open actual PNG.** Do NOT review code alone. Render first if figures do not exist.
2. Check against Core Standards in SKILL.md and [plot-standards.md](plot-standards.md) quick reject checklist. Fix the first problem found immediately — do NOT batch multiple issues, as one fix may resolve others.
3. Quick sanity glance — do values, units, sign conventions, and spatial patterns look physically plausible?
   - Temperature in K or °C, not raw integers or implausible ranges (e.g., 500 K surface temp)
   - Precipitation non-negative; wind speed non-negative
   - Cyclonic vorticity sign correct per hemisphere (negative in NH for ζ)
   - Spatial patterns make physical sense (warm SST in tropics, cold at poles, westerlies in mid-latitudes)
   - Anomaly magnitudes reasonable for the variable and time scale
4. Classify:
   - Visual-only → `REVISE` plotting code only.
   - Data/diagnostic issue → `REVISE` compute code + replot.
   - Unresolved science question → `BLOCKED` — state what needs the user's decision.
   - No issue → `PASS`.
5. After `REVISE`: re-render only affected figures, re-open PNG, continue until `PASS` or `BLOCKED`.
6. If 2 rounds fail on the same issue → stop patching. Reassess the approach — recheck data, coords, units, dtypes from scratch.
7. Max 10 RR iterations per figure. If still not `PASS`, surface remaining issues and hand off.

## Quick Fixes

| Defect | Fix |
|---|---|
| Colorbar overlaps axis | `shrink=0.8`, `pad=0.08`, or switch `orientation` |
| Panel label clips | `ax.text(-0.02, 1.03, ...)` with `transform=ax.transAxes` |
| White seam at 180° | Add `transform=ccrs.PlateCarree()`, use `ax.set_global()` |
| Ticks overlap | Reduce font; `MaxNLocator(nbins=5)` or `FixedLocator` |
| Stippling invisible | Increase marker size or switch to `hatches=["..."]` |
| Stippling too dense | Increase subsampling `[::4]` or reduce marker size; high-res data needs more skipping |
| Legend obscures data | `bbox_to_anchor=(1.02, 1)` outside axes |
| Colorbar washed out | Tighten `levels` to 2nd–98th percentile of the data range |
| Quiver too dense or sparse | Adjust skip: 1° → 3–5, 0.25° → 8–15, 2.5° → 1–2; larger domain needs more |
| Quiver key bad position or magnitude | Place outside axes at top-left: `quiverkey(Q, 0.0, 1.04, ref_val, "X m/s", labelpos="E", coordinates="axes")`. Ref magnitude = round number near **median** speed |
| Subplots too tight | `constrained_layout_pads(hspace=0.08, wspace=0.08)` |
| Gridline labels overlap at edges | `gl.top_labels = False; gl.right_labels = False` |
| Anomaly not centered at 0 | Use symmetric `levels`: e.g., `np.arange(-5, 5.5, 0.5)` with diverging cmap |
| Colorbar label missing units | Add `cbar.set_label("Variable (units)", fontsize=10)` |
| Cross-section y-axis not inverted | `ax.invert_yaxis()` or `ax.set_ylim(1000, 100)` |
