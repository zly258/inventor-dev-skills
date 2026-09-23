---
name: drawing-views
description: Create and lay out Inventor drawing sheets and base, projected, section, auxiliary, and detail views.
---

# Drawing Views

## Entry point

~~~csharp
if (app.ActiveDocument is not DrawingDocument drawing)
{
    throw new InvalidOperationException("A drawing document must be active.");
}

Sheet sheet = drawing.ActiveSheet;
DrawingViews views = sheet.DrawingViews;
~~~

Keep model-space geometry and sheet-space layout separate.

## Base view

DrawingViews.AddBaseView creates the primary model view from a Document.

Inputs include:
- model Document;
- sheet Point2d position;
- scale;
- ViewOrientationTypeEnum;
- DrawingViewStyleEnum;
- optional model view/representation options.

Use the actual model Document when possible instead of reopening the model by filename.

## Projected views

Use DrawingViews.AddProjectedView with:
- parent DrawingView;
- target sheet position;
- view style;
- optional scale.

Projected view orientation is derived from its position relative to the parent. Use deterministic layout coordinates.

## Section views

A section view requires:
- parent DrawingView;
- section-line DrawingSketch;
- target position;
- style and optional scale/depth settings.

Inventor 2025.1 introduced AddSectionView2 with additional options such as target sheet placement. If supporting older versions, keep this behind a compatibility layer and fall back to AddSectionView.

## Detail views

DrawingViews.AddDetailView requires the parent sheet to be active.

Before creating a detail view:
1. activate the sheet;
2. define the fence center/corners and radius/rectangle;
3. choose attach intent when required;
4. choose scale;
5. place the result away from existing views.

## View styles

Common styles include:
- hidden line;
- hidden line removed;
- shaded;
- shaded with hidden line;
- from-base style where supported.

Choose style from the drawing purpose. Do not globally force one style on every drawing.

## Scale selection

Do not choose scale by trial-and-error API calls.

Compute:
1. model/view range;
2. available sheet rectangle;
3. margin/reserved zones;
4. candidate standard scales;
5. largest scale that fits all required views.

Use one coherent scale policy and only override section/detail views when necessary.

## Layout strategy

For standard orthographic drawings:
1. reserve title block/border areas only if they exist;
2. place the base view;
3. place projected views using fixed directional relationships;
4. compute each view bounding box;
5. resolve overlaps by moving whole view groups;
6. place section/detail views in remaining regions;
7. only then create dimensions and annotations.

Do not place dimensions first and then move views underneath them.

## Four-view pattern

A useful deterministic layout for automation is:
- front/base;
- top;
- side;
- isometric.

The isometric view is primarily for shape recognition and should usually carry fewer manufacturing dimensions than orthographic views.

## View bounds

Use DrawingView position/size/range information to create a plain DTO:

~~~text
ViewBox
  Id
  CenterX
  CenterY
  Left
  Right
  Bottom
  Top
  Scale
  Orientation
~~~

Run layout algorithms on DTOs rather than repeatedly querying COM properties.

## Representation context

For assemblies and sheet metal, explicitly select the intended:
- model state;
- design view representation;
- positional representation;
- folded/flat pattern state.

A visually correct view with the wrong representation is still an incorrect drawing.

## Performance

Create all required views first, then wait/update at a controlled boundary before extracting final drawing curves for annotation.

Do not enumerate DrawingCurves repeatedly while Inventor is still regenerating the view.
