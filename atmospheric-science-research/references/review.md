# Review & Revision (RR)

Bounded loop that catches visual defects and obvious data-sanity issues. Not a full scientific re-audit.

## Loop

1. Render a 300 dpi PNG (plot script without `--final`) and open it; zoom in on small text, the key, and stippling.
2. Check layout: panel balance, colorbar proportion, overlap, clipping, whitespace.
3. Check against `plot-standards.md` and the Checklist below; any item not met means `REVISE`.
4. Check physics: value range, units, sign convention, pressure-axis orientation, spatial pattern.
5. Classify:
   - `PASS` — nothing remains.
   - `REVISE` — list every visible issue, batch all fixes that need no new information, re-render once, start the next iteration.
   - `BLOCKED` — needs a user decision, a missing input or dependency, or the output is unreadable.

Every figure gets at least one RR iteration before `PASS`; max 5 iterations per figure. If not `PASS` by then, or the same issue persists after two fix attempts, classify `BLOCKED` and record the remaining issues for the README.

Even when no rule fails, revise if the figure would look out of place in a top atmospheric-science journal: cluttered, unbalanced, inconsistently styled, or hard to read.

## Checklist

| Area | Pass when |
|---|---|
| Text | Arial or the project font; units in mathtext superscripts; nothing clipped or below 7 pt; no suptitle, caption, data stamp, or explanatory text |
| Panels | Bold letters at upper-left; short centered descriptions; letters, descriptions, and key on one baseline |
| Colormap | `cmaps` with explicit `levels`; sequential for absolute fields, 0-centered diverging for relative ones; no rainbow maps; `extend` set; comparable panels share `levels`, `cmap`, `extend` |
| Maps | `transform` on every Cartopy call; coastlines; lat/lon labels; no dateline seam or blank border |
| Tibet | Physical mask at every level; grey polygon at 700 hPa and below |
| Vectors | Readable density; one key at top-right with its tip on the right edge, mathtext units, and the field-arrow style; panels sharing a key share `ref_magnitude` and `ref_length` |
| Significance | Visible, subordinate to the field, no dark patches |
| Layout | Balanced panels; narrow colorbar no longer than its panel group; no overlap |
| Physics | Plausible values, correct units and sign convention, pressure axis inverted |

## Quick Fixes

Starting points; tune against the rendered PNG.

| Defect | First fix |
|---|---|
| Colorbar overlaps or dominates | `shrink=0.5–0.8`, `pad=0.06–0.10`, or a dedicated `GridSpec` slot |
| Ticks overlap | Fewer ticks via `MaxNLocator`; shrink the font only down to the minimum |
| Key arrow off the right edge | Redo `fig.canvas.draw()` and `key.x = 1.0 - key.patch.get_width()` after the last layout change |
| Vectors too dense / sparse | Lower / raise `density` |
| Stippling invisible / too dense | Change marker size, alpha, or the subsampling step |
| Legend hides data | Move it outside the axes with `bbox_to_anchor` |
| Layout still wrong | Change figure geometry or the `GridSpec`; code parameters are only estimates |
