# Review

`RR` = review & revision. Open real figures, check against plot standards and physical sense, patch minimum code, repeat until `PASS` or `BLOCKED`.

## Steps

1. Open the actual `PNG`. Do not review by code alone. Render first if figures do not exist.
2. Check visually against `plot-standards.md`. On finding a problem, patch immediately — do not batch.
3. Run physical-sense checks (below).
4. Classify:
   - visual-only → `REVISE` plotting code only
   - data/diagnostic issue → `REVISE` compute code + replot
   - unresolved science → `BLOCKED`
   - no issue → `PASS`
5. After `REVISE`, re-render only affected figures, re-open, and continue until `PASS` or `BLOCKED`.
6. If 3 rounds fail on the same issue, reassess the approach.

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

Vector fields: direction physically plausible, no single wildly large arrow, reference arrow labeled with units.
