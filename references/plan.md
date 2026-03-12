# Plan

Use this file only when the task needs a new workflow or a scientific-method change.
If the task is clearly `Execute-lite` or `RR-only`, skip this file.

## Goal

Define the execution path, not the implementation.

## What to do

1. Read the user goal, existing `README.md`, and current project structure if they exist.
2. Inspect candidate inputs and path structure quickly, only to the depth needed for design.
   - `NetCDF`: use `ncdump -h`
   - `GRIB`: inspect metadata with `cfgrib` or another equally light existing probe
   - confirm variables, dimensions, coordinates, units, time coverage, region, and missing-data conventions
3. Analyze the task and design a scientifically grounded solution.
   - assess whether the data and metadata are complete enough for a defensible method
   - identify the scientific target, the required preprocessing, the main analysis or statistics, the core diagnostics, and a sound computational methodology
   - state the key scientific and technical assumptions explicitly
   - if more than one defensible interpretation remains, present the tradeoff briefly instead of choosing silently
   - when multi-agent execution is available and the work can be split cleanly, split the plan into a `compute agent` path and a `plot agent` path with a clear handoff through reusable outputs
4. Stop and ask the user for clarification before `Execute` if any ambiguity remains in the data, method, diagnosis, or interpretation.
   - Don't assume. Don't hide confusion. Surface tradeoffs
   - prefer multiple-choice when helpful
   - if the scientific path is already clear, continue directly to `Execute`
   - prefer a single clarification stop; if the first answer still leaves a real scientific ambiguity, keep any follow-up narrowly focused on unblocking the method
5. Write or update a short Chinese `README.md` with [references/readme-template.md](references/readme-template.md).

## Plan rules

- Keep the plan short, practical, and execution-facing.
- Do not expand `Plan` into full implementation.
- Surface real uncertainty early; do not guess silently when the method is unclear.
- In the plan-stage `README.md`, describe the actual project structure if one already exists; otherwise keep `路径结构和实现功能` at the workflow level and fill exact file names after final outputs are ready. Leave `图片简述` and `后续版本补充` pending until outputs exist.
