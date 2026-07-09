---
name: spaceclaim
description: Generate, debug, and run Ansys SpaceClaim/Discovery automated modeling workflows from user geometry requirements. Use when Codex needs to turn a requested CAD shape, modeling procedure, geometry edit, boolean workflow, import/export task, or simulation-prep CAD operation into SpaceClaim API scripts, IronPython code, external CMD/Python/PowerShell launchers, or reusable automated modeling workflows.
---

# SpaceClaim

Use this skill to convert user modeling requirements into SpaceClaim automation code.

## Core Workflow

1. Translate the user's request into neutral modeling intent:
   - Coordinate system, units, reference planes, and origin conventions.
   - Bodies, sketches, profiles, features, imports, edits, booleans, groups, names, and exports.
   - Fixed dimensions or optional parameters, depending on what the user asked for.
   - Required outputs such as `.scdoc`, `.stp`, metadata, logs, or screenshots.
2. Treat paths as configurable placeholders:
   - `<WORKSPACE_ROOT>`: project root.
   - `<SCDM_INSTALL_DIR>`: folder containing `SpaceClaim.exe` and `SpaceClaim.Api.V*.dll`.
   - `<BASE_SCDOC>`: optional source `.scdoc`.
   - `<OUTPUT_DIR>`: generated output directory.
3. Discover `<SCDM_INSTALL_DIR>` before hardcoding it:
   - Scan fixed drives for `Program Files/ANSYS Inc/v*/scdm`.
   - Accept a candidate only if it contains `SpaceClaim.exe` and `SpaceClaim.Api.V*.dll`.
   - For each candidate, detect the available `SpaceClaim.Api.V*.dll` files and parse the numeric API version after `V`.
   - Prefer the candidate with the highest API version number by default.
   - If multiple candidates have the same highest API version or the user requests a specific Ansys version, present the candidates and ask which install to use.
4. Prefer SpaceClaim API calls over UI-recorded commands for fragile automation.
5. Run generated modeling code inside SpaceClaim with `Application.RunScript(scriptPath)`.
6. Launch SpaceClaim externally with:
   - `Api.Initialize()`
   - `Session.Start(StartupOptions)`
   - `Api.AttachToSession(session)`
   - `Application.RunScript(scriptPath)`
7. Before generating or adapting a Python launcher, check the user's shell environment:
   - Run `python --version` or `py -3.11 --version`.
   - If Python 3.11 is missing, tell the user to install Python 3.11 before relying on the generated Python command.
   - Keep this as a Codex/user-facing environment notice; do not embed Python installation guidance in the generated `.py` launcher itself.
8. Prefer a Python-first CMD entry for generated launchers:
   - `python run_spaceclaim_<task>.py` is the normal external command.
   - The Python launcher discovers SpaceClaim installs, lets the user select an install when multiple exist, and calls the PowerShell API bridge.
   - A `.bat` wrapper may call Python and print a clear Python 3.11 installation prompt if `python`/`py -3.11` is missing.
9. Keep PowerShell as the API bridge because it can load the selected `SpaceClaim.Api.V*.dll` directly without requiring a C# build.
10. Save native documents, export exchange files, and write metadata/error logs to `<OUTPUT_DIR>` when useful.

## Code Generation Guidance

- Generate the simplest SpaceClaim script that faithfully creates or modifies the requested geometry.
- Ask for clarification only when geometry, units, coordinate references, or output requirements are genuinely ambiguous.
- Do not force parameterization; use explicit dimensions when the user asks for a one-off model, and expose parameters when iteration, optimization, or variants are requested.
- Keep project-specific geometry confidential in reusable skill text, filenames, and prompts.
- Use construction geometry for clipping, layout, or references unless the user explicitly asks for physical solids.
- Name important bodies predictably for later material assignment, selection, or simulation import.
- When using a base `.scdoc`, open or import it first and operate on that document rather than recreating unrelated geometry.
- Write code that reports useful metadata such as body count, measured quantities, output paths, and boolean failures.

## Boolean And Robustness Rules

- Use the selected installed SpaceClaim API version's `Geometry/Profile/Modeler.Body` APIs where practical.
- For repeated features, generate one body or profile at a time and log indices/parameters.
- Boolean operations can fail with sparse `General Failure` messages.
- Prefer sequential unite/subtract operations with per-tool error logging over one large boolean.
- If a merge fails, keep unmerged bodies and record the failure instead of deleting geometry.
- If a subtract fails, keep the target body and report the failed tool body names.
- Include a dry-run mode that verifies script paths and API DLL paths without starting SpaceClaim when practical.

## Outputs

- Save `.scdoc` for later manual inspection and editing when requested or useful.
- Export `.stp` or another requested neutral format for downstream tools.
- Write metadata JSON when the script generates measurable geometry or runs many operations.
- Write an error file or launcher log when SpaceClaim/API execution fails.

## When Debugging

Read [references/spaceclaim-notes.md](references/spaceclaim-notes.md) for launcher patterns, Ansys install discovery, error handling, and SpaceClaim API pitfalls.
