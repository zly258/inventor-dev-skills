---
name: drawing-dimensions
description: Create stable automatic Inventor drawing dimensions using GeometryIntent, GeneralDimensions, deterministic classification, layout, and deduplication.
---

# Drawing Dimensions

Automatic dimensioning should be deterministic. Do not build a large candidate pool and repeatedly try random placements until something succeeds.

## Main APIs

For a sheet:

~~~csharp
DrawingDimensions drawingDimensions = sheet.DrawingDimensions;
GeneralDimensions general = drawingDimensions.GeneralDimensions;
~~~

GeneralDimensions provides creation methods such as:
- AddLinear / AddLinear2;
- AddAngular;
- AddDiameter;
- AddRadius;
- foreshortened variants where appropriate.

Use Sheet.CreateGeometryIntent to build intents from drawing geometry.

## Recommended pipeline

Use one pass per view:

1. extract drawing geometry once;
2. normalize geometry into plain DTOs;
3. classify lines/arcs/circles/endpoints;
4. map drawing entities back to model semantics when useful;
5. generate the required dimension set;
6. deduplicate;
7. classify each dimension to top/bottom/left/right;
8. assign dimension layers/offset levels;
9. solve spacing and collisions;
10. create dimensions in one controlled COM pass;
11. validate attached/orphaned state.

This is faster and more predictable than insert-fail-move-retry loops.

## Geometry extraction DTO

A useful normalized record can contain:

~~~text
DrawingEntity
  ViewId
  SourceKey
  CurveType
  StartX / StartY
  EndX / EndY
  CenterX / CenterY
  Radius
  DirectionX / DirectionY
  ModelFeatureType
  Visible
~~~

Keep COM DrawingCurve objects only long enough to create final GeometryIntent objects.

## Linear dimensions

Create GeometryIntent objects for the required points/curves and then call AddLinear/AddLinear2.

Choose the DimensionType deliberately:
- horizontal;
- vertical;
- aligned;
- arc/chord-specific cases where supported.

Do not infer orientation only from the final text position.

## Diameter and radius

For circular drawing geometry:
- classify whether the manufacturing intent is diameter or radius;
- create an intent on the arc/circle;
- place text outside dense geometry;
- avoid duplicating a hole note and a diameter dimension when the drawing standard does not require both.

GeneralDimensions.AddDiameter and AddRadius should receive a valid GeometryIntent created from the sheet.

## Stepped shaft rule

A stepped shaft often cannot be dimensioned adequately by reading only high-level model feature parameters.

For the longitudinal view:
1. detect axial direction;
2. collect shoulder locations projected onto the shaft axis;
3. cluster equal locations within tolerance;
4. sort unique stations;
5. create dimensions between adjacent stations for each step length;
6. create one overall length dimension from first to last station;
7. place segment dimensions on the first layer and the overall dimension farther outside.

This produces the common "each section length + total length" manufacturing pattern.

## Baseline and ordinate dimensions

Use baseline/ordinate dimension APIs when the drawing standard or part geometry benefits from a common datum.

Choose datum explicitly:
- functional mounting face;
- centerline;
- primary machined datum;
- defined origin.

Do not turn every chain into ordinate dimensions automatically.

## Dimension completeness

A complete dimension set normally combines:
- overall extents;
- local feature sizes;
- feature locations;
- diameters/radii;
- hole/thread semantics;
- angles where meaningful;
- repeated/pattern information;
- datums/tolerances when required by the task.

Geometry-driven rules are required for relationships that are visible in the final view but not represented as one convenient model feature.

## Four-side classification

Classify dimensions by where they should be placed relative to the view:
- horizontal lengths -> top or bottom;
- vertical lengths -> left or right;
- diameters/radii -> nearest low-density side;
- angular dimensions -> local open sector;
- overall dimensions -> outer layers.

Keep a per-side ordered list and assign offsets by layer.

## Spacing

Use configurable values for:
- view-to-first-dimension gap;
- dimension-to-dimension gap;
- text clearance;
- leader clearance;
- minimum spacing from neighboring views.

All spacing values must be converted through the drawing document's units.

## Deduplication

Create a canonical key before inserting a dimension.

Examples:
- linear: source A + source B + dimension type;
- diameter: source circle/hole identity;
- radius: source arc identity;
- overall: view + axis + extreme stations.

Normalize A/B ordering for symmetric keys.

Also compare existing generated dimensions using AttributeSets/reference data when rerunning automation.

## Avoiding weak dimensions

Reject or deprioritize:
- silhouette/tangent edges that are unstable for the intended measurement;
- tiny cosmetic geometry;
- hidden geometry unless specifically required;
- duplicated projected edges;
- dimensions that repeat information already conveyed by a hole/thread note.

## Collision handling

Do not delete and recreate repeatedly.

Before COM insertion:
1. estimate text/dimension envelope;
2. reserve view boxes;
3. reserve already-planned dimensions;
4. place to a deterministic free layer;
5. only use a bounded local adjustment if collision remains.

## Validation after creation

For every created dimension record:
- Attached;
- ModelValue;
- text origin;
- owning view/intent;
- AttributeSet generation ID;
- whether value was overridden.

Do not silently use OverrideModelValue to make an incorrect geometric dimension "look right."

## Failure handling

If one dimension cannot be created:
- identify the semantic dimension;
- report source entities/intents;
- log HRESULT;
- continue only when the missing dimension is non-critical;
- return a final completeness report.

A drawing command should know exactly which required dimensions were produced and which were not.
