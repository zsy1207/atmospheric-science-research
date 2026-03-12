# Review

`RR` = `review & revision`.
Use this file when figures already exist or the task is mainly to inspect and revise outputs.
Its job is to inspect real figures against `plot-standards.md`, check basic physical sense, validate any changed upstream outputs, and enter an immediate fix-and-rerender loop until the figure passes or a real scientific ambiguity blocks the task.

## RR steps

1. Open the actual output `PNG`. Do not review by code, filename, or directory listing alone. If the figures do not exist yet, render them first.
2. Check them visually against `plot-standards.md`.
   - once a concrete problem is confirmed, do not wait to collect every other issue first
   - move directly into the minimum relevant patch for that problem
3. Run a basic physical-sense check:
   - units and variable type match
   - sign, range, main anomaly centers, and gradient directions are not obviously wrong
   - vector direction, density, and colormap are reasonable
   - diagnostics and statistics implied by the current processed outputs and compute scripts are reasonable
4. Classify the issue:
   - visual-only -> `REVISE` and patch only the figure-side plotting code, such as `figN/plot_*.py` or the existing project equivalent
   - data or diagnostic issue -> `REVISE` and patch the affected compute code or upstream processed outputs, plus the corresponding plotting code
   - unresolved science -> `BLOCKED`
   - no remaining issue -> `PASS`
5. If the result is `REVISE`, patch immediately and re-render only the affected figure and any directly dependent preview or delivery exports, not the whole set.
6. Re-open the new output and continue issue-by-issue until the result is `PASS` or `BLOCKED`.

## RR focus

Prioritize these checks:
- text, ticks, legends, and colormap do not overlap or get clipped
- colormap scale ranges and intervals are scientific, and vector density is reasonable
- diagnostic and statistical methods are reasonable
- axis labels, units, and colorbars are complete
- map projection, coastlines, and boundary strategy are correct
- the figure is not crowded, misleading, or visually inconsistent
- the result is clean, readable, and journal-grade
- the exported file actually opens and was checked as a rendered figure, not inferred from code alone

Use `BLOCKED` when the computation method itself is ambiguous or the required change would clearly alter the scientific conclusion.
Use `PASS` only after the rerendered output has been re-opened and checked.

## README update

After the figure passes, update `README.md`:
- sync the path-structure section if scripts or outputs changed
- update `图片简述` if the figure meaning or appearance changed
- append one concise note in `后续版本补充`
