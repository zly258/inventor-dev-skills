---
name: part-features
description: Create and edit Inventor part features including extrude, revolve, sweep, loft, hole, fillet, chamfer, shell, mirror, and patterns.
---

# Part Features

Use this skill after the sketch/profile/reference geometry has already been validated.

## Main entry point

For a PartDocument:

~~~csharp
PartComponentDefinition definition = part.ComponentDefinition;
PartFeatures features = definition.Features;
~~~

Important feature collections include:
- ExtrudeFeatures
- RevolveFeatures
- SweepFeatures
- LoftFeatures
- HoleFeatures
- FilletFeatures
- ChamferFeatures
- ShellFeatures
- RectangularPatternFeatures
- CircularPatternFeatures
- MirrorFeatures

## General creation pattern

Inventor feature APIs commonly follow this pattern:

1. obtain the feature collection;
2. create a feature definition/input object when the API provides one;
3. set operation, extent, direction, taper, participants, or other options;
4. validate all referenced geometry;
5. call Add once;
6. inspect the resulting feature and update status.

Do not repeatedly call Add with slightly different random inputs when the geometry is invalid.

## Extrude

Typical dependencies:
- Profile from a PlanarSketch;
- operation such as join, cut, intersect, or new body;
- one-side, symmetric, through-all, to-face, or between extent depending on the API/version.

Use named expressions/parameters for distances when the feature is intended to be parametric.

Before a cut extrusion, verify that the profile intersects the intended body.

## Revolve

A revolve needs:
- a valid profile;
- a valid linear axis or sketch line;
- an operation;
- an angle/full revolution definition.

Do not infer the axis from sketch curve order. Resolve it by role/name/reference.

## Sweep

Sweep uses an Inventor.Path, not System.IO.Path.

This is a critical namespace collision:

~~~csharp
using InventorPath = Inventor.Path;
using IOPath = System.IO.Path;
~~~

A sweep requires:
- profile;
- path;
- orientation/twist rules as required;
- valid continuity and intersection conditions.

Build the path from intended sketch/3D sketch entities rather than storing transient edge indexes.

## Loft

Validate:
- section ordering;
- profile compatibility;
- rail/centerline references;
- closed/open section consistency;
- operation type.

A loft failure is often caused by inconsistent section topology, not by the Add call itself.

## Hole

Prefer HoleFeatures over manually extruding circles when the model semantically contains holes. Hole features preserve manufacturing intent and support drawing hole/thread notes better.

Handle separately:
- placement definition;
- diameter;
- depth/through-all extent;
- counterbore/countersink;
- thread/tapped settings;
- termination.

Do not convert every hole into anonymous cut geometry if downstream drawings/BOM/manufacturing logic depends on hole semantics.

## Fillet and chamfer

Select intended edges semantically. Avoid Edges.Item(n).

For generated edge groups:
1. classify by owning feature;
2. classify by geometry type/direction/radius;
3. use tolerances;
4. create the fillet/chamfer from the resolved set.

If one edge fails, report which semantic edge set failed instead of swallowing the COM error.

## Shell

Before shell creation:
- verify body is solid;
- identify remove faces semantically;
- check thickness against local geometry size;
- use UnitsOfMeasure for thickness.

Thin-wall failures should return diagnostics that identify the requested thickness and selected faces.

## Patterns and mirrors

Pattern/mirror features should reference source features, not duplicate their construction logic.

Prefer:
- RectangularPatternFeatures for linear repeated features;
- CircularPatternFeatures for rotational repetition;
- MirrorFeatures for feature/body symmetry.

Keep direction axes/work geometry stable and named.

## Multi-body parts

Be explicit about whether an operation:
- joins an existing body;
- cuts selected participants;
- creates a new body.

When features support participant bodies, set them deliberately. Do not rely on whichever body Inventor happens to select by default.

## Error diagnostics

When feature creation fails, log:
- feature type;
- operation;
- profile/path/reference identity;
- extent/distance/angle in both requested and database units;
- participant body count;
- source feature names;
- COM HRESULT and message.

A useful error says why the feature could not be built, not only "Build failed."
