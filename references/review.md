# Review & Revision (RR)

RR is a **figure-focused** review loop. The goal is to catch and fix visual defects, not to re-audit the science.

## Steps

1. **Open actual PNG.** Do NOT review code alone. Render first if figures do not exist.
2. Check against HARD RULES in SKILL.md and [plot-standards.md](plot-standards.md) quick reject checklist. Patch immediately on finding a problem — do NOT batch multiple issues.
3. Quick sanity glance: do values, units, sign conventions, and spatial patterns look physically plausible? (e.g., temperature in K not raw integers, precipitation non-negative, cyclonic vorticity sign correct per hemisphere.)
4. Classify:
   - Visual-only → `REVISE` plotting code only.
   - Data/diagnostic issue → `REVISE` compute code + replot.
   - Unresolved science question → `BLOCKED` — state what needs the user's decision.
   - No issue → `PASS`.
5. After `REVISE`: re-render only affected figures, re-open PNG, continue until `PASS` or `BLOCKED`.
6. If 2 rounds fail on the same issue → reassess approach, do NOT keep patching blindly.
7. Max 10 RR iterations per figure. If still not `PASS`, surface remaining issues and hand off.

## Quick fixes

| Defect | Fix |
|---|---|
| Colorbar overlaps axis | `shrink=0.8`, `pad=0.08`, or switch `orientation` |
| Panel label clips | `ax.text(-0.02, 1.03, ...)` with `transform=ax.transAxes` |
| White seam at 180° | Add `transform=ccrs.PlateCarree()`, use `ax.set_global()` |
| Ticks overlap | Reduce font; `MaxNLocator(nbins=5)` or `FixedLocator` |
| Stippling invisible | Increase marker size or switch to `hatches=["..."]` |
| Stippling too dense (solid black patches) | Increase subsampling `[::4]` or reduce marker size; high-res data needs more skipping |
| Legend obscures data | `bbox_to_anchor=(1.02, 1)` outside axes |
| Colorbar washed out | Tighten `levels` to 2nd–98th percentile |
| Quiver too dense or sparse | Adjust skip: 1° → 3–5, 0.25° → 8–15, 2.5° → 1–2; larger domain needs more skipping |
| Quiver key wrong position or bad magnitude | White opaque box flush against lower-right border: `FancyBboxPatch((0.84,0.0),0.16,0.1,...)` + `quiverkey(Q, 0.92, 0.04, ref_val, ...)`. Arrow must fit inside box. Ref magnitude = round number near **median** speed, not max |
| Subplots too tight | `constrained_layout_pads(hspace=0.08, wspace=0.08)` |
