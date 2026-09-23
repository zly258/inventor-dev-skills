---
name: drawing-sheets-tables
description: Create Inventor drawing documents and sheets, use templates/styles, and manage borders, title blocks, sketched symbols, parts lists, hole tables, and revision tables.
---

# Drawing Documents, Sheets, and Tables

Use this skill for the drawing document/sheet structure around the actual model views and dimensions.

## Create a drawing

Prefer a known drawing template.

General pattern:

~~~csharp
string template = app.FileManager.GetTemplateFile(
    DocumentTypeEnum.kDrawingDocumentObject);

DrawingDocument drawing = (DrawingDocument)app.Documents.Add(
    DocumentTypeEnum.kDrawingDocumentObject,
    template,
    true);
~~~

Inventor 2025 also provides FileManager.GetTemplateFileWithOptions for more explicit template-source options.

For production systems, do not assume every workstation has the same hard-coded template path. Resolve it through Inventor/project configuration or a product configuration layer.

## Template policy

A drawing template can already define:
- drafting standard;
- styles;
- layers;
- sheet size;
- border;
- title block;
- sketched symbols;
- revision table setup;
- default drawing resources.

Prefer using a correct template over recreating all drawing resources in code on every document.

## Sheets

DrawingDocument.Sheets contains Sheet objects.

A Sheet provides access to areas such as:
- DrawingViews;
- DrawingDimensions;
- PartsLists;
- RevisionTables;
- SketchedSymbols;
- drawing sketches;
- center marks/centerlines/notes/symbol collections;
- TitleBlock;
- border/sheet properties.

Before an API requires an active sheet, call sheet.Activate deliberately.

## Sheet size and orientation

Treat width, height, standard size, and orientation as layout inputs.

Build a sheet-layout DTO with:
- usable rectangle;
- margins;
- title-block reserved area;
- view regions;
- table regions.

Do not hard-code A3/A2 coordinates inside view-placement commands.

## Border and title block

Title blocks and borders belong to drawing resources/templates.

When automation must replace/insert them:
- find the intended definition by stable configured name;
- remove/replace only when the workflow owns that resource;
- populate linked iProperties instead of drawing static duplicate text where possible.

Do not rebuild a title block from raw lines/text every time a drawing is generated.

## Styles

DrawingDocument.StylesManager exposes drawing styles and layers.

Prefer:
- active standard;
- existing named styles;
- ObjectDefaults styles;
- local cached styles where appropriate.

Do not hard-code lineweight, font, text height, precision, arrow style, and layer color throughout dimension/annotation code.

If a required style is missing, report it clearly or use a deliberate configured fallback.

## Parts lists

PartsLists are sheet objects backed by assembly/model BOM data.

Place tables after major view layout is known.

Reserve a table region and then:
- create/select the correct PartsList;
- apply intended PartsListStyle;
- choose columns;
- sort/renumber if required;
- keep balloon item numbering consistent.

Do not treat PartsList as an independent database.

## Hole tables

Use HoleTables for hole-heavy plate/flange drawings when the drawing standard benefits from coordinate/tabular hole definition.

Before creation:
- define the source view;
- define origin/datum;
- decide grouping/tag policy;
- reserve table/tag space.

Avoid redundant full coordinate dimensioning when a hole table already communicates the same information.

## Revision tables

RevisionTables are drawing documentation objects.

Keep revision data ownership explicit:
- user/PDM-managed;
- Add-In-managed;
- template-managed.

Do not overwrite revision history as a side effect of regenerating dimensions.

## Sketched symbols

Use SketchedSymbols for reusable standard symbols/callouts that are not better represented by a dedicated Inventor annotation type.

Prefer definitions from the template. Keep prompt/property values separate from geometry.

## Layout order

Recommended drawing generation order:

1. create drawing from template;
2. configure sheets;
3. reserve border/title-block/table regions;
4. create model views;
5. update views;
6. dimension;
7. annotate;
8. create BOM/hole/revision tables;
9. final collision check;
10. final update/export.

This prevents late table insertion from pushing dimensions into unreadable areas.

## Up-to-date state

Sheet.Status can indicate that view/precise display processing is not complete.

Before geometry-sensitive operations such as final dimension extraction or plotting/export, ensure the relevant sheet/views are up to date.

Do not read final drawing geometry while the sheet is still regenerating in the background.
