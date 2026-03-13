# Review

`RR` = review & revision. Open real figures, check against plot standards and physical sense, patch minimum code, repeat until `PASS` or `BLOCKED`.

## Steps

1. Open the actual `PNG`. Do not review by code alone. Render first if figures do not exist.
2. Check visually against `plot-standards.md`. On finding a problem, patch immediately — do not batch.
3. Run physical-sense checks (below).
4. Classify:
   - visual-only → `REVISE` plotting code only
   - data/diagnostic issue → `REVISE` compute code + replot
   - unresolved science → `BLOCKED` — state what needs the user's decision
   - no issue → `PASS`
5. After `REVISE`, re-render only affected figures, re-open, and continue until `PASS` or `BLOCKED`.
6. If 3 rounds fail on the same issue, reassess the approach — do not keep patching blindly.

### Exit criteria

- `PASS`: all plot-standards items clear, physical-sense checks pass, no visual defects.
- `BLOCKED`: the fix requires a scientific decision or method change that only the user can make. State the question clearly.
- Max 5 RR iterations per figure. If still not `PASS`, surface remaining issues and hand off.

## Common defects & quick fixes

| Defect | Quick fix |
|---|---|
| Colorbar text overlaps axis | Move colorbar: `shrink=0.8`, `pad=0.08`, or switch `orientation` |
| Panel labels `(a)` clip at edge | Use `ax.text(-0.02, 1.03, ...)` with `transform=ax.transAxes`, not `ax.set_title` |
| White seam at 180° on global map | Add `transform=ccrs.PlateCarree()` to plot call; use `ax.set_global()` |
| Tick labels overlap on small panels | Reduce font size; use `MaxNLocator(nbins=5)` or `mticker.FixedLocator` |
| Faint stippling invisible in print | Increase marker size or switch to hatching: `hatches=["..."]` |
| Legend obscures data | `ax.legend(loc="upper left", bbox_to_anchor=(1.02, 1))` outside axes |
| Colorbar range too wide (washed out) | Tighten `levels` to the 2nd–98th percentile of the data |
| Quiver arrows too dense | Increase skip: `Q = ax.quiver(lon[::n], lat[::n], u[::n,::n], v[::n,::n])` |
| Subplot spacing too tight | `fig.set_constrained_layout_pads(hspace=0.08, wspace=0.08)` |

## Physical-sense checks

General: units match labels/colorbar, sign convention correct, value range reasonable, no artifacts (stripes, checkerboard, unexpected NaN), land/ocean masking correct.

| Variable | Key checks |
|---|---|
| Temperature | K (200–320) or °C (−70–+50); anomaly ±0.5–10 K |
| Precipitation | Non-negative; wet regions (ITCZ, storm tracks) in expected locations |
| Wind | Speed 0–80 m/s; jet 30–70 m/s at 200 hPa; surface < 25 m/s climatology |
| Geopotential | Smooth; ~5500 m at 500 hPa mid-lat; equator-to-pole gradient |
| SST | −2 to +35°C; warm pool and cold tongue in correct locations |
| MSLP | 960–1050 hPa; lows in storm tracks, highs in subtropics |
| OLR | 100–300 W/m²; low over deep convection, high over deserts |
| Vorticity | Cyclonic sign matches hemisphere (negative NH, positive SH) |
| EOF | Leading mode matches known patterns (e.g., ENSO for tropical Pacific SST) |
| Relative humidity | 0–100%; high near ITCZ and frontal zones |
| Vertical velocity (omega) | Negative = upward (Pa/s); −1 to +1 Pa/s typical; strong upward in convective regions |
| Cloud cover | 0–1 or 0–100%; high over ITCZ, storm tracks, stratus decks |
| Soil moisture | Non-negative; dry in deserts, wet in tropics |

Vector fields: direction physically plausible, no single wildly large arrow, reference arrow labeled with units.

### Season and region sanity

- NH summer (JJA): ITCZ shifted north, monsoon active, jet stream weakened and poleward.
- NH winter (DJF): strong jet, deep Aleutian/Icelandic lows, subtropical highs weaker.
- Anomaly sign should be consistent with known modes (e.g., El Niño → warm Niño 3.4, positive SOI → La Niña).
