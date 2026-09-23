---
name: geometry-selection-references
description: Select Inventor entities, create transient geometry, map model/drawing geometry, and preserve references across operations.
---

# Geometry, Selection, and References

## Selection

For interactive commands, use Inventor selection filters rather than accepting arbitrary objects.

Typical sources:
- Application.CommandManager.Pick for a single filtered pick;
- Document.SelectSet for the current selection;
- InteractionEvents/SelectEvents for richer interactive tools.

Always validate both the selected object type and the owning document.

## Object collections

Many Inventor APIs accept ObjectCollection. Create it through TransientObjects:

~~~csharp
ObjectCollection edges = app.TransientObjects.CreateObjectCollection();

foreach (Edge edge in selectedEdges)
{
    edges.Add(edge);
}
~~~

Do not pass a normal List<T> where the COM API expects ObjectCollection.

## Model lookup

Useful ComponentDefinition APIs include:
- FindUsingPoint;
- FindUsingRay;
- FindUsingVector;
- RangeBox / PreciseRangeBox where supported.

Use tolerance-aware geometry tests. Never compare floating-point coordinates with exact equality.

## Reference keys

For entities that need to be found again later, prefer Inventor reference keys over storing raw COM object references.

General pattern:
1. obtain the entity's reference key with GetReferenceKey;
2. persist the byte data/context needed by ReferenceKeyManager;
3. later bind the key back through the document's ReferenceKeyManager;
4. validate that the rebound object still has the expected semantic role.

Reference keys improve persistence but are not a substitute for semantic validation after major topology changes.

## AttributeSets

AttributeSets are useful for product-owned metadata on API objects that support them.

Use a unique namespace/set name for your Add-In. Store only compact identity/metadata, not large serialized application state.

Good uses:
- mark generated features;
- store a stable product ID;
- associate a drawing annotation with a generation rule.

## Persistent topology strategy

For robust automation, use a layered identity strategy:

1. product-owned feature ID / AttributeSet;
2. ReferenceKey;
3. owning feature and geometry type;
4. geometric signature such as center, radius, normal, direction, or bounds;
5. final tolerance-based search.

Do not identify a face only as Faces.Item(3). Collection order can change after edits.

## Model-to-drawing mapping

To dimension model geometry in a drawing:
- start with the DrawingView;
- use DrawingView.DrawingCurves(modelGeometry) when model geometry is known;
- otherwise enumerate drawing curves and classify them;
- create GeometryIntent objects on the sheet;
- pass those intents to drawing dimension/annotation APIs.

Keep model-space and sheet-space coordinates separate.
