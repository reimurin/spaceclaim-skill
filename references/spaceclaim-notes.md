# SpaceClaim Automation Notes

## API Entry Points

- `Application.RunScript(path)` executes `.py` or `.scscript` inside SpaceClaim.
- External process control sequence:
  - `Api.Initialize()`
  - `Session.Start(StartupOptions)`
  - `Api.AttachToSession(session)`
  - `Application.RunScript(scriptPath)`
- `StartupOptions.ExecutableFolder` should point to `<SCDM_INSTALL_DIR>`.
- `StartupOptions.ShowApplication = false` supports hidden runs.
- `Session.Stop()` can close SpaceClaim after completion.

## Launcher Pattern

- Local files may not reveal a stable command-line `RunScript` argument for `SpaceClaim.exe`.
- PowerShell can load the selected `<SCDM_INSTALL_DIR>/SpaceClaim.Api.V*.dll` directly.
- Do not assume Python, Git, or .NET SDK exists in `PATH`.
- When using this skill to generate or adapt a Python launcher, first check `python --version` or `py -3.11 --version` in the user's shell.
- If Python 3.11 is missing, tell the user to install Python 3.11 before relying on the generated `python run_*.py` command.
- Keep Python installation guidance in the Codex/user-facing skill workflow, not inside the generated `.py` launcher.
- For external CMD usage, use this wrapper stack:
  - Python `.py`: normal user command for orchestration and SpaceClaim install discovery.
  - BAT `.bat`: optional Windows convenience wrapper.
  - PowerShell `.ps1`: real API launcher that loads the SpaceClaim API DLL.
- Include a dry-run mode that verifies script and API DLL paths without starting SpaceClaim when practical.
- Log launcher activity to `<OUTPUT_DIR>/spaceclaim_launcher.log`.

## Discovering Ansys/SpaceClaim Installs

- Do not hardcode the install folder in reusable automation.
- Scan common Windows install roots across fixed drives, such as:
  - `<DRIVE>:/Program Files/ANSYS Inc/v*/scdm`
  - `<DRIVE>:/Program Files (x86)/ANSYS Inc/v*/scdm`
- Treat a folder as a SpaceClaim candidate only when both exist:
  - `SpaceClaim.exe`
  - `SpaceClaim.Api.V*.dll`
- Parse the numeric API version from `SpaceClaim.Api.V*.dll`, such as `SpaceClaim.Api.V23.dll` -> `23`.
- Parse the `vXXX` folder name as an Ansys release hint, but prefer the API DLL version for launcher selection.
- If multiple API DLLs exist in one install folder, use the highest numeric API version unless the user specifies otherwise.
- If multiple candidates exist, prefer the candidate with the highest API version number by default.
- If multiple candidates share the highest API version or the user requests a specific version/install, present all candidates with version folder, API version, and path, then ask the user to choose.
- If one candidate exists, use it by default but report the chosen folder.
- If no candidates exist, ask the user for `<SCDM_INSTALL_DIR>`.

Example PowerShell API selection inside one `scdm` folder:

```powershell
$apiDll = Get-ChildItem $scdm -Filter "SpaceClaim.Api.V*.dll" |
    ForEach-Object {
        if ($_.Name -match "SpaceClaim\.Api\.V(\d+)\.dll") {
            [pscustomobject]@{ Version = [int]$Matches[1]; Path = $_.FullName }
        }
    } |
    Sort-Object Version -Descending |
    Select-Object -First 1
```

## Modeling Code Notes

- Convert the user's request into explicit planes, axes, dimensions, profiles, extrusions, sweeps, revolves, patterns, transforms, and boolean operations.
- Use construction boundaries for clipping or layout unless the user explicitly asks for physical boundary solids.
- Avoid overlapping repeated feature families unless the intended model requires intersections or lattice behavior.
- Name important bodies and components predictably.
- Keep units explicit and convert consistently at the API boundary.
- Keep generated files and logs under `<OUTPUT_DIR>`.

## Body Creation And Booleans

- Use the selected installed SpaceClaim API version's `Geometry/Profile/Modeler.Body` APIs where practical.
- `Body.ExtrudeProfile` can create solid bodies from profiles.
- `DesignBody.MassProperties.Mass` may be usable as volume when density is set to unity, but verify the local API/model convention.
- `Document.SaveAs` saves `.scdoc`.
- `Part.Export(PartExportFormat.Step, path, true, null)` exports STEP.
- `Body.Unite` and subtract operations may throw generic `General Failure`.
- Prefer sequential unite/subtract with per-tool error logging over one large boolean.
- If an operation fails, keep the recoverable geometry and write the failure to metadata.

## Error Handling

- SpaceClaim may show sparse stack traces only.
- Write detailed errors to `<OUTPUT_DIR>/<case_name>_error.txt`.
- Write geometry metadata to `<OUTPUT_DIR>/<case_name>_metadata.json`.
- Include input values, generated body counts, measured quantities, body names, merge errors, and subtract errors in metadata when relevant.

## Path Neutrality

- Do not bake drive letters into reusable docs or skills.
- Use placeholders:
  - `<WORKSPACE_ROOT>`
  - `<SCDM_INSTALL_DIR>`
  - `<BASE_SCDOC>`
  - `<OUTPUT_DIR>`
- Keep project-specific mappings only as examples outside reusable skill text.
- It is acceptable to describe the discovery pattern `*/Program Files/ANSYS Inc/v*/scdm`; the actual drive letter and version must be discovered or selected per machine.
