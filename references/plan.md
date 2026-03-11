# Plan

Use this file only when the task needs a new workflow or a scientific-method change.
If the task is clearly `Execute-lite` or `RR-only`, skip this file.

## Goal

Define the execution path, not the implementation.

## What to do

1. Read the user goal, existing `README.md`, and current project structure if they exist.
2. Inspect candidate inputs and path structure quickly, only to the depth needed for design.
   - do not write processed outputs there
   - `NetCDF`: use `ncdump -h`
   - `GRIB`: inspect metadata with `cfgrib` if available, otherwise use another equally light probe already available in the environment or project
   - confirm variables, dimensions, coordinates, units, time coverage, region, and missing-data conventions
   - determine whether the repo already has an established layout such as `scripts/`, `figures/`, `paper_figs/`, or a processed-data directory, and reuse it when possible
3. Analyze the task and design a scientifically grounded solution.
   - assess whether the data and metadata are complete enough for a defensible method
   - identify the scientific target, the required preprocessing, the main analysis or statistics, the core diagnostics, and a sound computational methodology
   - state the key scientific and technical assumptions explicitly.
   - if more than one defensible interpretation remains, present the tradeoff briefly instead of choosing silently
   - define the minimal execution path using the current project structure first; if the repo has no usable layout, fall back to `data_processed/`, `figN/plot_*.py`, and optional `figN/compute_*.py`
   - when multi-agent execution is available and the work can be split cleanly, split the plan into a `compute agent` path and a `plot agent` path with a clear handoff through reusable outputs
4. Decide whether a clarification stop is needed before `Execute`.
   - if a remaining scientific ambiguity would change the method, diagnostic, or interpretation, stop and ask the user concise questions; prefer multiple-choice when helpful
   - if the scientific path is already clear, continue directly to `Execute`
   - prefer a single clarification stop; if the first answer still leaves a real scientific ambiguity, keep any follow-up narrowly focused on unblocking the method
5. Write or update a short Chinese `README.md` with [references/readme-template.md](references/readme-template.md).

## Plan rules

- Keep the plan short, practical, and execution-facing.
- If one stop is needed, ask only the minimum questions needed to unblock execution. Multiple rounds are allowed only if the first answer still leaves a real scientific ambiguity.
- Do not expand `Plan` into full implementation.
- Do not do heavy computation in `Plan`.
- Surface real uncertainty early; do not guess silently when the method is unclear.
- In the plan-stage `README.md`, describe the actual project structure if one already exists; otherwise keep `路径结构和实现功能` at the workflow level and fill exact file names after final outputs are ready. Leave `图片简述` and `后续版本补充` pending until outputs exist.
