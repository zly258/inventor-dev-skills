---
name: drawing-annotations
description: Create Inventor drawing annotations including center marks, centerlines, hole/thread notes, leader notes, balloons, symbols, and tables.
---

# Drawing Annotations

Dimensions and annotations are different layers of drawing automation. Create views first, dimensions second, then annotations/tables so layout can reserve space consistently.

## Common annotation families

Typical Inventor drawing annotation areas include:
- Centermarks;
- Centerlines;
- HoleThreadNotes;
- GeneralNotes;
- LeaderNotes;
- Balloons;
- SurfaceTextureSymbols;
- FeatureControlFrames;
- WeldSymbols;
- HoleTables;
- RevisionTables and other drawing tables where required.

Use the active drawing standard/style instead of reproducing visual formatting manually.

## Center marks and centerlines

For visible circular geometry:
- create center marks when required by the drawing standard;
- create centerlines for aligned/coaxial geometry;
- deduplicate by referenced circle/arc or axis group.

Do not place a center mark on every circular-looking edge without checking whether it represents a hole, shaft, cosmetic arc, or repeated projected edge.

## Hole and thread notes

Prefer HoleThreadNotes when the model contains actual hole/thread semantics.

Advantages:
- note content is derived from model data;
- changes to hole/thread features can update the drawing note;
- manufacturing intent is clearer than a plain text leader.

Do not replace a semantic hole/thread note with hard-coded text unless the API cannot represent the required note.

## General and leader notes

Use plain notes for information that is not a model-driven dimension:
- manufacturing instruction;
- finish instruction;
- process note;
- local callout.

For leaders:
1. identify the target GeometryIntent;
2. compute a text anchor in an open region;
3. create a short, clear leader path;
4. keep leader crossings to a minimum.

## Balloons

Balloons are BOM-linked annotations.

Balloons.Add uses an ObjectCollection for leader points. The final item must be a GeometryIntent that identifies the attached geometry.

Conceptually:

~~~csharp
ObjectCollection leader = app.TransientObjects.CreateObjectCollection();
leader.Add(sheetPointA);
leader.Add(sheetPointB);
leader.Add(sheet.CreateGeometryIntent(targetCurve));

Balloon balloon = sheet.Balloons.Add(leader);
~~~

Use actual model/BOM identity. Do not invent balloon numbers independently from the BOM.

## Surface finish and GD&T

For SurfaceTextureSymbols and FeatureControlFrames:
- use drawing styles/standards;
- attach to semantic geometry;
- preserve datum relationships;
- keep symbol generation separate from normal dimensional tolerances.

Do not encode GD&T as arbitrary GeneralNote text when a dedicated Inventor annotation object exists.

## Hole tables

Hole tables can be useful when a plate contains many repeated holes.

Before choosing a hole table:
- verify the view is appropriate;
- define origin/datum;
- decide whether similar holes should be grouped;
- ensure tags do not collide with dimensions.

Do not simultaneously add exhaustive coordinate dimensions and a full hole table unless the drawing standard requires both.

## Annotation placement

Classify leader-style annotations by the target location relative to the view:
- upper-left;
- upper-right;
- lower-left;
- lower-right;
- top/bottom/left/right bands.

A simple radial/four-side spreading strategy is often more stable than repeated collision-driven retries.

Recommended sequence:
1. get target point;
2. classify quadrant/side;
3. allocate next anchor on that side;
4. route leader outward;
5. apply a bounded collision adjustment.

## Styles and layers

Prefer:
- active standard;
- named dimension/annotation styles;
- named layers.

Do not hard-code font, lineweight, color, and text height throughout business logic.

When a required named style is unavailable, either:
- fall back deliberately to the active standard; or
- return a clear configuration error.

## Rerun behavior

Generated annotations should be idempotent.

Store product metadata using AttributeSets or a stable mapping so rerunning the command can:
- update existing generated items;
- remove obsolete generated items;
- avoid duplicates;
- leave user-created annotations untouched.

## Validation

After annotation:
- confirm Attached where applicable;
- check leader target validity;
- check whether text overlaps the owning view;
- check whether the annotation is outside the sheet;
- verify no duplicate semantic note was created.

A successful API call is not enough; the drawing must remain readable.
