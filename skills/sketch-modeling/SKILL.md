---
name: sketch-modeling
description: Create robust Inventor 2D sketches, work geometry, constraints, profiles, and sketch-based modeling inputs.
---

# Sketch Modeling

## Sketch plane

Create planar sketches from a planar Face or WorkPlane:

~~~csharp
PartComponentDefinition definition = part.ComponentDefinition;
PlanarSketch sketch = definition.Sketches.Add(definition.WorkPlanes[3], false);
~~~

When orientation matters, use PlanarSketches.AddWithOrientation instead of accepting an arbitrary default X/Y direction.

For face-based sketches, decide deliberately whether UseFaceEdges should be true. Automatically projecting every face edge often creates noisy, fragile sketches.

## Work geometry

Use:
- WorkPlanes;
- WorkAxes;
- WorkPoints;
- UserCoordinateSystems

to define stable modeling references.

Prefer a named/reference-driven work plane over "the first planar face" when the operation must survive model changes.

## Sketch curves

Common collections include:
- SketchLines;
- SketchCircles;
- SketchArcs;
- SketchSplines;
- SketchEllipses.

Create sketch geometry in sketch coordinates and convert model/sketch coordinates explicitly when required.

## Constraints

Use geometric constraints to encode design intent:
- horizontal / vertical;
- parallel / perpendicular;
- coincident;
- tangent;
- concentric;
- equal;
- symmetry where appropriate.

Use dimensional constraints for driving sizes.

Do not over-constrain a sketch with redundant generated constraints.

## Profiles

A closed-looking sketch is not automatically a valid solid profile.

For solid features use Profiles.AddForSolid. Inspect the returned profile paths when multiple closed regions exist.

For surface operations use Profiles.AddForSurface when an open profile is valid.

## Robust profile rules

Before feature creation:
1. verify expected loops exist;
2. reject zero-length curves;
3. merge or avoid duplicate coincident geometry;
4. use tolerances for endpoint matching;
5. know whether inner loops are islands/holes;
6. explicitly select the intended ProfilePath set when the sketch contains multiple regions.

## Parametric modeling

Prefer named parameters and expressions for dimensions that define design intent. Do not scatter literal database-unit values across feature code.

Keep a clean split:
- sketch construction;
- constraint/dimension creation;
- profile extraction;
- feature creation.

This makes failed profiles much easier to diagnose.
