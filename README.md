<h1 align="center">Inventor Dev Skills</h1>

<p align="center">Modular agent skill library for Autodesk Inventor C# in-process Add-In development.</p>

<p align="center">
  <a href="./README.md">English</a> | <a href="./README.zh-CN.md">简体中文</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/C%23-.NET%208%20%2F%20.NET%204.8-3776AB?style=flat-square" alt="C# .NET 8 and .NET 4.8"> <img src="https://img.shields.io/badge/Autodesk%20Inventor-Windows%20x64-4B5563?style=flat-square" alt="Autodesk Inventor on Windows x64"> <a href="./SKILL.md"><img src="https://img.shields.io/badge/Skills-1%20Master%20%2B%2020%20Focused-4F46E5?style=flat-square" alt="1 Master Skill and 20 Focused Skills"></a> <a href="./LICENSE"><img src="https://img.shields.io/badge/License-Apache%202.0-22C55E?style=flat-square" alt="License: Apache 2.0"></a>
</p>

Inventor Dev Skills is a structured collection of implementation rules for **in-process Autodesk Inventor C# Add-In development**. It turns recurring Inventor failure modes—wrong target frameworks, `Any CPU` builds, `.addin` manifest mismatches, namespace collisions, centimeter database unit bugs, off-thread COM access, stale geometry references, and non-deterministic drawing dimensions—into reusable, task-scoped agent skills.

This repository covers in-process Inventor C# Add-Ins only. It explicitly excludes MCP servers, standalone automation EXEs, AI chat/workflow architectures, and the Fusion 360 API.

## Highlights

| Highlight | Why it matters |
| --- | --- |
| Master router plus 20 focused skills | Load [`SKILL.md`](./SKILL.md) first and pull in only the domain skills needed for the current task, keeping agent context small and preventing unrelated APIs from competing for attention. |
| Dual runtime baseline coverage | Separates Inventor 2025+ (`.NET 8` / `net8.0-windows`, Visual Studio 2022 17.8+) from Inventor 2024 and earlier (`net48` or release-matched SDK target) so projects do not force the wrong runtime. |
| COM interop and namespace guardrails | Enforces explicit aliases for conflicting types (`System.IO.Path` vs `Inventor.Path`, `Inventor.Application` vs `System.Windows.Application`), main-thread COM discipline, and structured HRESULT logging with no empty `catch` blocks. |
| Deterministic modeling and drawing rules | Promotes single-pass COM extraction into plain C# DTOs, semantic identity (`AttributeSets`, `ReferenceKey`) over fragile collection indexes, and deterministic automatic drawing dimension pipelines. |
| Registry-free deployment and diagnostics | Documents supported Windows `.addin` manifest locations, PowerShell string-interpolation traps, and failure-signature troubleshooting across build, load, COM, BOM, and deployment stages. |

## Architecture

[`SKILL.md`](./SKILL.md) acts as the repository-level master router that establishes mandatory rules and dispatches tasks to 20 focused skills organized under `skills/`:

```text
┌────────────────────────────────────────────────────────────────────────────────┐
│                         Master Skill (SKILL.md)                                │
│    Scope Guard | Mandatory Rules | Version Baseline | Task-to-Skill Router     │
└───────────────────────────────────────┬────────────────────────────────────────┘
                                        │ load only task-relevant skills
                                        ▼
┌────────────────────────────────────────────────────────────────────────────────┐
│                          skills/ (20 Focused Skills)                           │
│                                                                                │
│  ┌───────────────────────────────────┐  ┌───────────────────────────────────┐  │
│  │ Add-In Foundation & Interop       │  │ Core Modeling & Assemblies        │  │
│  │ - api-map                         │  │ - documents-units-transactions    │  │
│  │ - project-setup                   │  │ - geometry-selection-references   │  │
│  │ - addin-lifecycle                 │  │ - sketch-modeling                 │  │
│  │ - installation-deployment         │  │ - part-features                   │  │
│  │ - ribbon-events                   │  │ - sheet-metal                     │  │
│  │ - com-interop-conflicts           │  │ - assembly                        │  │
│  └───────────────────────────────────┘  └───────────────────────────────────┘  │
│                                                                                │
│  ┌───────────────────────────────────┐  ┌───────────────────────────────────┐  │
│  │ Drawings & Annotations            │  │ Data, I/O & Reliability           │  │
│  │ - drawing-sheets-tables           │  │ - bom-properties                  │  │
│  │ - drawing-views                   │  │ - import-export                   │  │
│  │ - drawing-dimensions              │  │ - performance-reliability         │  │
│  │ - drawing-annotations             │  │ - troubleshooting                 │  │
│  └───────────────────────────────────┘  └───────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────────┘
```

## Quick Install

Clone the repository locally so your coding agent or workspace can reference [`SKILL.md`](./SKILL.md) and the focused skill files under `skills/`:

```bash
git clone https://github.com/zly258/inventor-dev-skills.git
```

Prerequisites for target Inventor Add-In projects:

- Windows (`x64`) with Autodesk Inventor and the matching Inventor Add-In SDK/template installed.
- **Inventor 2025 and later:** Visual Studio 2022 17.8+, `net8.0-windows`, `x64`, registry-free `.addin` deployment.
- **Inventor 2024 and earlier:** the .NET framework supported by the target release's Autodesk SDK/template (commonly `net48`).

## Quick Start

1. Load the repository-level master skill [`SKILL.md`](./SKILL.md) first so scope boundaries and mandatory engineering rules are active.
2. Add only the focused skills required for your task. Typical combinations:

| Goal | Skills to load alongside `SKILL.md` |
| --- | --- |
| Build a new Inventor Add-In | `project-setup`, `addin-lifecycle`, `installation-deployment`, `ribbon-events`, `com-interop-conflicts` |
| Create a parametric part | `documents-units-transactions`, `geometry-selection-references`, `sketch-modeling`, `part-features` |
| Create an assembly command | `documents-units-transactions`, `geometry-selection-references`, `assembly` |
| Automate a manufacturing drawing | `drawing-sheets-tables`, `drawing-views`, `drawing-dimensions`, `drawing-annotations`, `bom-properties`, `performance-reliability` |
| Diagnose a loader or build failure | `project-setup`, `addin-lifecycle`, `installation-deployment`, `com-interop-conflicts`, `troubleshooting` |

3. Before editing an existing Add-In project, verify the target Inventor version, target framework, `x64` build configuration, interop reference, `AddInServer` GUID, `.addin` manifest, and deploy directory.
4. Before marking work complete, verify a clean `x64` build with no new warnings, clean Add-In loading on Inventor startup, wrong-document-type handling, transaction rollback on failure, and idempotent reruns.

## Compatibility and Core Rules

### Runtime compatibility

- **Inventor 2025+:** Autodesk moved Inventor Add-In templates to `.NET 8` (`net8.0-windows`). Build `x64` and deploy via registry-free `.addin` manifests.
- **Inventor 2024 and earlier:** Follow the Autodesk SDK/template framework for that release (`net48` is a common legacy baseline; do not force it onto newer releases without validation).
- **Multi-generation support:** Share core source where practical and produce target-specific Add-In builds and manifests.

### Explicit aliases for conflicting types

Qualify high-frequency collisions between `System.*`, WPF, and `Inventor.*`:

```csharp
using IOPath = System.IO.Path;
using IOFile = System.IO.File;
using InventorPath = Inventor.Path;
using InventorApplication = Inventor.Application;
using WpfApplication = System.Windows.Application;
using InventorTransaction = Inventor.Transaction;
```

Use `System.IO.Path` for filesystem paths and `Inventor.Path` for modeling or sweep paths.

### Keep Inventor COM objects on the Inventor thread

Do not access live Inventor COM objects inside `Task.Run`:

```csharp
// Avoid
await Task.Run(() => part.ComponentDefinition.Features.Count);
```

Instead, follow the four-step extraction pipeline:

1. Read Inventor state on the Inventor thread.
2. Convert it into plain C# DTOs.
3. Perform pure computation off-thread if useful.
4. Return to the Inventor context to apply mutations.

### Respect database units

Inventor database units use **centimeter** for length, **radian** for angle, and **kilogram** for mass—regardless of document UI display units. Convert via `UnitsOfMeasure` at API boundaries.

### Deterministic automatic drawing dimensions

Avoid unbounded candidate-pool retry loops. Use a deterministic single-pass pipeline:

```text
extract drawing geometry once
    -> normalize DTOs
    -> generate semantic required dimensions
    -> deduplicate
    -> classify top/bottom/left/right
    -> assign offset layers
    -> resolve spacing/collisions
    -> create dimensions once
    -> validate
```

For a stepped shaft, detect shoulder stations from drawing geometry, create each adjacent section length, then create one overall length dimension.

### Error handling and data-layer separation

- **No empty `catch` blocks:** Always record the API operation, `HRESULT`, document, semantic entity, and key inputs when an Inventor call fails.
- **Separate model data from drawing presentation:** Material, calculated mass, assembly BOM, drawing `PartsList`, and cell overrides are distinct layers; a visible table cell is not automatically a writable source property.

### Registry-free `.addin` deployment

Common supported deployment locations on Windows:

- `%ALLUSERSPROFILE%\Autodesk\Inventor Addins\`
- `%PROGRAMFILES%\Autodesk\Inventor 20xx\Bin\Addins\` (version-dependent all-user manifests on Inventor 2024+)
- `%APPDATA%\Autodesk\Inventor 20xx\Addins\`
- `%APPDATA%\Autodesk\ApplicationPlugins`

A simple per-user version-specific deployment can keep the manifest, Add-In DLL, and managed dependencies in one flat product directory.

When writing PowerShell deployment scripts, brace variables that precede a colon:

```powershell
# Wrong
throw "Deployment validation failed for Inventor $version: $required"

# Correct
throw "Deployment validation failed for Inventor ${version}: $required"
```

## Documentation

### Skill catalog

| Topic | What it covers | Link |
| --- | --- | --- |
| Master router | Scope boundaries, mandatory rules, version baseline, task routing | [SKILL.md](./SKILL.md) |
| API map | Quick Inventor object-model and interface entry points | [skills/api-map/SKILL.md](./skills/api-map/SKILL.md) |
| Project setup | Target framework, `x64`, interop references, WPF, build output | [skills/project-setup/SKILL.md](./skills/project-setup/SKILL.md) |
| Add-In lifecycle | `ApplicationAddInServer`, `Activate`, `Deactivate`, event lifetime | [skills/addin-lifecycle/SKILL.md](./skills/addin-lifecycle/SKILL.md) |
| Installation and deployment | `.addin` manifest, install paths, load rules, PowerShell | [skills/installation-deployment/SKILL.md](./skills/installation-deployment/SKILL.md) |
| COM interop and conflicts | `Path`/`File`/`Application`/`Point`/`Transaction` conflicts, COM threading | [skills/com-interop-conflicts/SKILL.md](./skills/com-interop-conflicts/SKILL.md) |
| Documents, units, transactions | Documents, database units, transient objects, updates, save rules | [skills/documents-units-transactions/SKILL.md](./skills/documents-units-transactions/SKILL.md) |
| Geometry, selection, references | `SelectSet`, `ObjectCollection`, `ReferenceKey`, `AttributeSets`, proxies | [skills/geometry-selection-references/SKILL.md](./skills/geometry-selection-references/SKILL.md) |
| Sketch modeling | Work geometry, 2D sketches, constraints, profiles | [skills/sketch-modeling/SKILL.md](./skills/sketch-modeling/SKILL.md) |
| Part features | Extrude, revolve, sweep, loft, hole, fillet, chamfer, shell, patterns | [skills/part-features/SKILL.md](./skills/part-features/SKILL.md) |
| Sheet metal | `SheetMetalComponentDefinition`, flat patterns, DXF/DWG manufacturing | [skills/sheet-metal/SKILL.md](./skills/sheet-metal/SKILL.md) |
| Assembly | Occurrences, transforms, proxies, constraints, traversal, BOM impact | [skills/assembly/SKILL.md](./skills/assembly/SKILL.md) |
| Drawing sheets and tables | Templates, sheets, styles, title blocks, parts lists, hole/revision tables | [skills/drawing-sheets-tables/SKILL.md](./skills/drawing-sheets-tables/SKILL.md) |
| Drawing views | Base, projected, section, auxiliary, and detail views, scale, layout | [skills/drawing-views/SKILL.md](./skills/drawing-views/SKILL.md) |
| Drawing dimensions | `GeometryIntent`, `GeneralDimensions`, deterministic dimension planning | [skills/drawing-dimensions/SKILL.md](./skills/drawing-dimensions/SKILL.md) |
| Drawing annotations | Center marks, centerlines, hole/thread notes, leaders, balloons, GD&T | [skills/drawing-annotations/SKILL.md](./skills/drawing-annotations/SKILL.md) |
| BOM and properties | BOM, `PartsList`, iProperties, material, mass, model-state scope | [skills/bom-properties/SKILL.md](./skills/bom-properties/SKILL.md) |
| Ribbon UI and events | Button definitions, Ribbon tabs/panels, event lifetime, WPF | [skills/ribbon-events/SKILL.md](./skills/ribbon-events/SKILL.md) |
| Import and export | `TranslatorAddIn`, STEP, DWG, DXF, PDF, flat-pattern export | [skills/import-export/SKILL.md](./skills/import-export/SKILL.md) |
| Performance and reliability | COM round-trips, batching, DTO architecture, transactions, idempotency | [skills/performance-reliability/SKILL.md](./skills/performance-reliability/SKILL.md) |
| Troubleshooting | Build, load, COM, modeling, drawing, BOM, and deployment failure signatures | [skills/troubleshooting/SKILL.md](./skills/troubleshooting/SKILL.md) |

### Official Autodesk references

- [Inventor 2026 API Help](https://help.autodesk.com/view/INVNTOR/2026/ENU/)
- [Creating an Add-In](https://help.autodesk.com/cloudhelp/2026/ENU/Inventor-API/files/CreatingAnAddIn_Overview.htm)
- [ApplicationAddInServer](https://help.autodesk.com/cloudhelp/2023/ENU/Inventor-API/files/ApplicationAddInServer.htm)
- [PartComponentDefinition](https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/PartComponentDefinition.htm)
- [DrawingViews](https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/DrawingViews.htm)
- [GeneralDimensions](https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/GeneralDimensions.htm)
- [UnitsOfMeasure](https://help.autodesk.com/cloudhelp/2023/ENU/Inventor-API/files/UnitsOfMeasure.htm)
- [TranslatorAddIn](https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/TranslatorAddIn.htm)

## License

Licensed under the Apache License 2.0. See [LICENSE](./LICENSE).

Autodesk and Inventor are trademarks of Autodesk, Inc. This repository is an independent developer resource and is not affiliated with or endorsed by Autodesk.
