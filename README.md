# spaceclaim-skill
An automation code generation and debugging skill of Codex.
Ansys SpaceClaim (SCDM) is a fast direct CAD modeling tool bundled within Ansys Discovery and the Ansys Workbench ecosystem, widely used for geometry creation, repair, cleanup, boolean editing, and pre-simulation CAD preparation. It exposes a complete .NET-based IronPython API for automated batch modeling, eliminating repetitive manual UI operations.
Skill Introduction
This skill converts user geometric modeling requirements into reusable SpaceClaim automated workflow code, supporting native IronPython scripts, external Python/PowerShell/CMD launchers, and batch geometry processing pipelines.
Core capabilities:
Auto-locate local Ansys SpaceClaim installation directories and auto-select the highest-version API DLL;
Generate robust API-first modeling scripts (sketch, solid, boolean, import/export, geometry repair, simulation prep);
Provide cross-shell launch wrappers (Python / BAT / PowerShell) to run scripts externally outside SpaceClaim;
Built-in error logging, dry-run verification, metadata export, and fault-tolerant boolean operation logic;
Batch export native .scdoc and neutral CAD formats (STEP) for downstream simulation workflows.

Ansys SpaceClaim（简称 SCDM）是集成于 Ansys Discovery、Workbench 体系的直接建模 CAD 工具，多用于几何建模、模型修复、几何清理、布尔编辑与仿真前处理。软件提供完整.NET 底层 IronPython 开放 API，可实现全流程自动化建模，替代重复手动界面操作。
工具简介
本模块可将用户几何建模需求一键转化为可复用的 SpaceClaim 自动化流程代码，支持原生 IronPython 脚本、外部 Python/PowerShell/CMD 启动器与批量几何处理流水线。
核心功能：
自动扫描本地 Ansys 安装目录，识别 SpaceClaim 程序并自动选用最高版本 API 动态库；
生成以 API 底层调用为核心的稳健建模代码，覆盖草图、实体、布尔运算、模型导入导出、几何修复、仿真预处理；
提供多终端启动封装脚本（Python/BAT/PowerShell），支持脱离软件界面外部批量运行；
内置运行日志、预校验干跑模式、元数据输出、容错型布尔运算失败处理逻辑；
批量输出原生.scdoc工程文件与 STEP 等通用中性格式，对接后续仿真流程。
