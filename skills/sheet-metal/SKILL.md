---
name: sheet-metal
description: Work with Inventor sheet metal component definitions, sheet metal features, flat patterns, and manufacturing export.
---

# Sheet Metal

Sheet metal is not just a normal PartComponentDefinition with different commands. Detect and use SheetMetalComponentDefinition explicitly.

## Detect sheet metal

~~~csharp
if (part.ComponentDefinition is not SheetMetalComponentDefinition sheetMetal)
{
    throw new InvalidOperationException("The active part is not a sheet metal document.");
}
~~~

SheetMetalComponentDefinition derives from PartComponentDefinition, but exposes sheet-metal-specific behavior and FlatPattern support.

## Core areas

Common API areas include:
- sheet metal styles and thickness;
- Face/Flange/Cut/Bend/Fold-related sheet metal features;
- punch features;
- A-side/orientation;
- Unfold / Unfold2;
- FlatPattern;
- flat-pattern export.

Do not model all sheet metal as generic extrudes if downstream flat patterns and manufacturing data matter.

## Flat pattern

Check HasFlatPattern before reading FlatPattern.

~~~csharp
if (!sheetMetal.HasFlatPattern)
{
    sheetMetal.Unfold();
}

FlatPattern flat = sheetMetal.FlatPattern;
~~~

If a specific base face is required, use the appropriate unfold workflow instead of assuming the automatic orientation is correct.

FlatPattern can be invalidated by upstream geometry changes. Revalidate it before export.

## Multi-body caution

A normal multi-body sheet metal part has restrictions around flat pattern creation. Do not assume one flat pattern can represent arbitrary multiple bodies in the same active model state.

If manufacturing requires separate flat patterns, prefer separate parts/model-state strategies consistent with Inventor's sheet metal rules.

## Thickness and style

Treat sheet metal thickness as design data controlled by the sheet metal definition/style. Avoid maintaining an unrelated duplicate "thickness" value in your own metadata.

When code changes style/thickness:
- update the model;
- validate bends/flanges;
- validate flat pattern;
- re-read extents.

## Flat-pattern geometry

Useful FlatPattern information includes:
- SurfaceBodies;
- TopFace;
- Width;
- Length where supported by the target API;
- work geometry;
- reference keys;
- model geometry version.

Use the flat pattern coordinate system for manufacturing extraction, not folded-model coordinates.

## DXF/DWG export

Inventor supports flat-pattern export through the flat pattern DataIO workflow and translator APIs.

For DataIO, the format string can control layers such as:
- outer profile;
- interior profiles;
- bend up/down;
- tangent geometry;
- arc centers;
- tool centers.

Do not hard-code a machine-specific format string in modeling code. Keep manufacturing export options in a dedicated configuration object.

## Export validation

Before exporting a flat pattern:
1. confirm HasFlatPattern;
2. ensure the document is updated;
3. verify no flat-pattern compute errors;
4. validate output path with System.IO.Path;
5. overwrite only when explicitly allowed;
6. report final extents and output file.

## Drawing integration

When creating drawing views of sheet metal, explicitly decide whether the drawing view represents:
- folded model;
- flat pattern.

Dimensions, bend notes, hole notes, and manufacturing annotations depend on that choice.
