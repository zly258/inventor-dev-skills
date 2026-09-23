# Inventor Dev Skills

A practical skill library for Autodesk Inventor secondary development with C#.

The repository is intentionally focused on **in-process Inventor Add-In development**: project setup, Add-In lifecycle, COM interoperability, modeling, assemblies, drawings, dimensions, annotations, BOM/iProperties, Ribbon UI, deployment, import/export, performance, and troubleshooting.

It does **not** include MCP servers, standalone automation EXEs, or AI chat/workflow architecture.

## Why this repository exists

Inventor C# development has many small failure modes that are easy to repeat across projects:

- the wrong .NET target for the Inventor release;
- Any CPU instead of x64;
- .addin manifest/GUID/path mismatches;
- System.IO.Path vs Inventor.Path;
- System.IO.File vs Inventor file-related objects;
- Inventor.Application vs System.Windows.Application;
- Inventor.Transaction vs System.Transactions.Transaction;
- raw millimeter values passed to APIs that use centimeter database units;
- stale Face/Edge references after model regeneration;
- empty catch blocks hiding COM failures;
- background Task.Run code touching live Inventor COM objects;
- automatic drawing dimensions implemented as slow retry loops;
- PartsList cells treated as if they were always writable model properties.

These skills turn those lessons into reusable implementation rules.

## Compatibility

### Inventor 2025 and later

Autodesk moved the Inventor Add-In templates to **.NET 8** starting with Inventor 2025.

Recommended baseline:

- Visual Studio 2022 17.8+
- net8.0-windows
- x64
- Autodesk Inventor Add-In template/SDK for the target release
- registry-free .addin deployment

### Inventor 2024 and earlier

Use the framework supported by the Autodesk SDK/template for the target release.

net48 is a common legacy baseline, but it should not be forced onto newer Inventor releases without validation.

If one product supports both runtime generations, share core source where practical and produce target-specific Add-In builds/manifests.

## How to use

Start with the repository-level master skill:

[SKILL.md](SKILL.md)

Then load only the focused skills needed for the task.

This keeps agent context small and prevents unrelated Inventor APIs from competing for attention.

## Skill catalog

| Skill | Purpose |
| --- | --- |
| [API map](skills/api-map/SKILL.md) | Quick Inventor object-model and interface map |
| [Project setup](skills/project-setup/SKILL.md) | Target framework, x64, interop references, WPF, build output |
| [Add-In lifecycle](skills/addin-lifecycle/SKILL.md) | ApplicationAddInServer, Activate, Deactivate, event lifetime |
| [Installation and deployment](skills/installation-deployment/SKILL.md) | .addin manifest, install paths, load rules, PowerShell |
| [COM interop and conflicts](skills/com-interop-conflicts/SKILL.md) | Path/File/Application/Point/Transaction conflicts, COM threading |
| [Documents, units, transactions](skills/documents-units-transactions/SKILL.md) | Documents, database units, transient objects, updates, save rules |
| [Geometry, selection, references](skills/geometry-selection-references/SKILL.md) | SelectSet, ObjectCollection, ReferenceKey, AttributeSets, proxies |
| [Sketch modeling](skills/sketch-modeling/SKILL.md) | Work geometry, 2D sketches, constraints, profiles |
| [Part features](skills/part-features/SKILL.md) | Extrude, revolve, sweep, loft, hole, fillet, chamfer, shell, patterns |
| [Sheet metal](skills/sheet-metal/SKILL.md) | SheetMetalComponentDefinition, flat patterns, DXF/DWG manufacturing |
| [Assembly](skills/assembly/SKILL.md) | Occurrences, transforms, proxies, constraints, traversal, BOM impact |
| [Drawing sheets and tables](skills/drawing-sheets-tables/SKILL.md) | Templates, sheets, styles, title blocks, parts lists, hole/revision tables |
| [Drawing views](skills/drawing-views/SKILL.md) | Base/projected/section/detail views, scale, layout, representations |
| [Drawing dimensions](skills/drawing-dimensions/SKILL.md) | GeometryIntent, GeneralDimensions, automatic dimension planning |
| [Drawing annotations](skills/drawing-annotations/SKILL.md) | Center marks, centerlines, hole/thread notes, leaders, balloons, GD&T |
| [BOM and properties](skills/bom-properties/SKILL.md) | BOM, PartsList, iProperties, material, mass, model-state scope |
| [Ribbon UI and events](skills/ribbon-events/SKILL.md) | Button definitions, Ribbon tabs/panels, event lifetime, WPF |
| [Import and export](skills/import-export/SKILL.md) | TranslatorAddIn, STEP, DWG, DXF, PDF, flat-pattern export |
| [Performance and reliability](skills/performance-reliability/SKILL.md) | COM round-trips, batching, DTO architecture, transactions, idempotency |
| [Troubleshooting](skills/troubleshooting/SKILL.md) | Build/load/COM/modeling/drawing/BOM/deployment failure signatures |

## Typical skill combinations

### Build an Inventor Add-In

Load:

- SKILL.md
- project-setup
- addin-lifecycle
- installation-deployment
- ribbon-events
- com-interop-conflicts

### Create a parametric part

Load:

- SKILL.md
- documents-units-transactions
- geometry-selection-references
- sketch-modeling
- part-features

### Create an assembly command

Load:

- SKILL.md
- documents-units-transactions
- geometry-selection-references
- assembly

### Automatically create a manufacturing drawing

Load:

- SKILL.md
- drawing-sheets-tables
- drawing-views
- drawing-dimensions
- drawing-annotations
- bom-properties
- performance-reliability

### Diagnose a loader/build failure

Load:

- SKILL.md
- project-setup
- addin-lifecycle
- installation-deployment
- com-interop-conflicts
- troubleshooting

## High-value rules

### Use explicit names for conflicting types

~~~csharp
using IOPath = System.IO.Path;
using IOFile = System.IO.File;
using InventorPath = Inventor.Path;
using InventorApplication = Inventor.Application;
using WpfApplication = System.Windows.Application;
using InventorTransaction = Inventor.Transaction;
~~~

Use System.IO.Path for filesystem paths and Inventor.Path for modeling/sweep paths.

### Keep Inventor COM objects on the Inventor thread

Do not do this:

~~~csharp
await Task.Run(() => part.ComponentDefinition.Features.Count);
~~~

Instead:

1. read Inventor state on the Inventor thread;
2. convert it into plain C# DTOs;
3. perform pure computation off-thread if useful;
4. return to the Inventor context for mutations.

### Respect database units

Inventor database units include:

- length: centimeter;
- angle: radian;
- mass: kilogram.

Use UnitsOfMeasure at API boundaries.

### Automatic drawing dimensions should be deterministic

Recommended architecture:

~~~text
extract drawing geometry once
    -> normalize DTOs
    -> generate semantic required dimensions
    -> deduplicate
    -> classify top/bottom/left/right
    -> assign offset layers
    -> resolve spacing/collisions
    -> create dimensions once
    -> validate
~~~

Do not use unbounded candidate-pool retries.

For a stepped shaft, detect shoulder stations from the drawing geometry, create each adjacent section length, then create one overall length dimension.

### Do not hide COM failures

Empty catch blocks are prohibited.

Capture:

- API operation;
- HRESULT;
- document;
- semantic entity;
- important inputs.

### Keep model data and drawing presentation separate

Material, calculated mass, assembly BOM, drawing PartsList, and cell overrides are different layers.

A visible table cell is not automatically a writable source property.

## Deployment notes

Autodesk recommends registry-free .addin deployment.

Common supported locations include:

- %ALLUSERSPROFILE%\Autodesk\Inventor Addins\
- %PROGRAMFILES%\Autodesk\Inventor 20xx\Bin\Addins\ for version-dependent all-user manifests on Inventor 2024+
- %APPDATA%\Autodesk\Inventor 20xx\Addins\
- %APPDATA%\Autodesk\ApplicationPlugins

A simple per-user version-specific deployment can keep the manifest, Add-In DLL, and managed dependencies in one flat product directory.

The deployment skill also documents the PowerShell interpolation trap:

~~~powershell
# Wrong
throw "Deployment validation failed for Inventor $version: $required"

# Correct
throw "Deployment validation failed for Inventor ${version}: $required"
~~~

## Official Autodesk references

- Inventor 2026 API Help: https://help.autodesk.com/view/INVNTOR/2026/ENU/
- Creating an Add-In: https://help.autodesk.com/cloudhelp/2026/ENU/Inventor-API/files/CreatingAnAddIn_Overview.htm
- ApplicationAddInServer: https://help.autodesk.com/cloudhelp/2023/ENU/Inventor-API/files/ApplicationAddInServer.htm
- PartComponentDefinition: https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/PartComponentDefinition.htm
- DrawingViews: https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/DrawingViews.htm
- GeneralDimensions: https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/GeneralDimensions.htm
- UnitsOfMeasure: https://help.autodesk.com/cloudhelp/2023/ENU/Inventor-API/files/UnitsOfMeasure.htm
- TranslatorAddIn: https://help.autodesk.com/cloudhelp/2025/ENU/Inventor-API/files/TranslatorAddIn.htm

## License

Apache License 2.0. See [LICENSE](LICENSE).

## Disclaimer

Autodesk and Inventor are trademarks of Autodesk, Inc. This repository is an independent developer resource and is not affiliated with or endorsed by Autodesk.
