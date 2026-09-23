---
name: inventor-dev
description: Master skill for Autodesk Inventor secondary development with C#. Routes Add-In, modeling, assembly, drawing, annotation, BOM, UI, deployment, and troubleshooting tasks to focused skills.
---

# Inventor C# Secondary Development

This is the master skill for this repository.

Use it for in-process Autodesk Inventor C# Add-In development only.

## Scope

Included:
- C# project and Inventor SDK setup;
- ApplicationAddInServer lifecycle;
- registry-free .addin deployment;
- COM interop and namespace conflicts;
- documents, units, transactions;
- geometry selection and persistent references;
- sketches and work geometry;
- part features;
- sheet metal;
- assemblies;
- drawing views;
- automatic dimensions;
- drawing annotations;
- BOM, PartsList, iProperties, material, and mass;
- Ribbon UI and events;
- import/export;
- performance and reliability;
- troubleshooting.

Excluded:
- MCP servers;
- standalone automation EXEs;
- AI chat architecture;
- Fusion 360 API.

## Load focused skills

Always load only the focused skills required for the task.

| Task | Skill |
| --- | --- |
| API object entry point / class lookup | skills/api-map/SKILL.md |
| target framework, references, x64, WPF | skills/project-setup/SKILL.md |
| Activate/Deactivate/AddInServer | skills/addin-lifecycle/SKILL.md |
| .addin/install/uninstall/deployment | skills/installation-deployment/SKILL.md |
| Path/File/Application/COM/thread conflicts | skills/com-interop-conflicts/SKILL.md |
| documents/units/transactions/save/update | skills/documents-units-transactions/SKILL.md |
| selection/reference keys/AttributeSets | skills/geometry-selection-references/SKILL.md |
| sketches/profiles/work planes | skills/sketch-modeling/SKILL.md |
| extrude/revolve/sweep/loft/hole/etc. | skills/part-features/SKILL.md |
| sheet metal/flat pattern | skills/sheet-metal/SKILL.md |
| occurrences/constraints/proxies | skills/assembly/SKILL.md |
| drawing view creation/layout | skills/drawing-views/SKILL.md |
| automatic dimensions | skills/drawing-dimensions/SKILL.md |
| notes/balloons/center marks/GD&T | skills/drawing-annotations/SKILL.md |
| BOM/PartsList/iProperties/material/mass | skills/bom-properties/SKILL.md |
| Ribbon/buttons/events/WPF | skills/ribbon-events/SKILL.md |
| STEP/DWG/DXF/PDF/translators | skills/import-export/SKILL.md |
| slow/unstable batch automation | skills/performance-reliability/SKILL.md |
| unknown build/load/API failure | skills/troubleshooting/SKILL.md |

## Version baseline

Autodesk Inventor 2025 and later moved Inventor Add-In templates to .NET 8.

For Inventor 2025+:
- prefer net8.0-windows;
- use Visual Studio 2022 17.8+;
- build x64;
- use the Autodesk Inventor Add-In template/SDK for the installed target version.

For Inventor 2024 and earlier:
- use the runtime/framework target supported by that release's Autodesk SDK/template;
- net48 is a common legacy target, but it must not be forced onto newer releases without validation.

If one product supports both runtime generations, share core source where practical but produce target-specific Add-In builds/manifests.

## Mandatory rules

### 1. Check document type

Never cast ActiveDocument blindly.

### 2. Respect Inventor database units

Do not assume millimeters because the UI displays millimeters.

Use UnitsOfMeasure. Database length is centimeter and database angle is radian.

### 3. Qualify conflicting types

High-frequency collisions:

~~~csharp
using IOPath = System.IO.Path;
using IOFile = System.IO.File;
using InventorPath = Inventor.Path;
using InventorApplication = Inventor.Application;
using WpfApplication = System.Windows.Application;
using InventorTransaction = Inventor.Transaction;
~~~

Use System.IO.Path for files and Inventor.Path for sweep/model paths.

### 4. Keep Inventor COM objects on the Inventor thread

Never put live Inventor API objects in Task.Run.

Extract plain DTOs first, compute off-thread only when useful, then apply mutations back through the Inventor context.

### 5. No empty catches

Never use catch { } around Inventor operations.

At minimum record operation, HRESULT, document, semantic entity, and inputs.

### 6. Prefer semantic identity over collection indexes

Do not persist Faces.Item(3), Edges.Item(8), Occurrences.Item(2), or similar as stable identity.

Prefer:
- AttributeSets/product IDs;
- ReferenceKey;
- owning feature;
- geometry signature;
- tolerance-based fallback.

### 7. Separate extraction, planning, and mutation

For complex automation:

~~~text
Inventor COM extraction
    -> plain C# DTOs
    -> deterministic analysis/layout
    -> final operation plan
    -> Inventor COM mutation
    -> one controlled update/validation
~~~

This architecture improves speed, determinism, diagnostics, and testability.

### 8. Drawing automation must be deterministic

For automatic dimensions:
- extract once;
- classify once;
- create semantic required dimensions;
- deduplicate;
- assign top/bottom/left/right;
- assign offset layers;
- solve spacing/collisions;
- commit once.

Do not use unbounded candidate-pool retries.

For a stepped shaft, dimension adjacent shoulder stations plus one overall length.

### 9. Keep model data and drawing presentation separate

Material, mass, BOM, PartsList, and displayed cells are not interchangeable.

Do not make a calculated PartsList value editable merely because it is visible.

### 10. Keep Add-In identity stable

The AddInServer Guid, .addin ClassId/ClientId, command InternalName values, and persisted UI identity must be managed deliberately.

## Before modifying a project

Inspect:
- target Inventor version;
- target framework;
- x64 setting;
- Inventor interop reference;
- existing AddInServer Guid;
- .addin manifest;
- deploy directory;
- current document/API layer;
- whether the project already has compatibility abstractions.

Preserve working architecture unless the task explicitly requires refactoring.

## Before considering a task complete

Validate:
- clean build;
- Add-In loads from a clean Inventor start;
- no new compiler warnings;
- wrong document type is handled;
- transaction aborts on failure;
- rerun does not duplicate generated artifacts;
- no empty catch blocks;
- no accidental Path/File/Application ambiguity;
- no background Inventor COM access;
- output is correct in the target Inventor version;
- deployment package contains the current DLL and dependencies.
