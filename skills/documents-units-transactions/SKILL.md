---
name: documents-units-transactions
description: Work safely with Inventor documents, database units, updates, save state, and transactions.
---

# Documents, Units, Updates, and Transactions

## Check document type before casting

~~~csharp
var doc = app.ActiveDocument;
if (doc is null || doc.DocumentType != DocumentTypeEnum.kPartDocumentObject)
{
    throw new InvalidOperationException("A part document must be active.");
}

var part = (PartDocument)doc;
~~~

Use the same pattern for AssemblyDocument and DrawingDocument. Do not assume ActiveDocument is non-null or of the expected type.

## Database units

Inventor API database units are independent of the user's display units.

Important database units:
- length: centimeter;
- angle: radian;
- mass: kilogram.

A raw numeric length of 100 means 100 cm, not 100 mm.

Use UnitsOfMeasure at API boundaries:

~~~csharp
static double MillimetersToDatabase(Document doc, double millimeters)
{
    return doc.UnitsOfMeasure.ConvertUnits(
        millimeters,
        UnitsTypeEnum.kMillimeterLengthUnits,
        UnitsTypeEnum.kDatabaseLengthUnits);
}
~~~

Use UnitsOfMeasure to evaluate user-entered expressions instead of manually stripping unit text.

## Transient objects

Use Application.TransientGeometry and Application.TransientObjects for temporary API values.

~~~csharp
var tg = app.TransientGeometry;
Point2d p2 = tg.CreatePoint2d(1.0, 2.0);
Point p3 = tg.CreatePoint(0.0, 0.0, 0.0);
Matrix transform = tg.CreateMatrix();

ObjectCollection objects = app.TransientObjects.CreateObjectCollection();
~~~

Coordinates still use database units.

## Transactions

Use an Inventor transaction around one logical mutation:

~~~csharp
Inventor.Transaction? transaction = null;

try
{
    transaction = app.TransactionManager.StartTransaction(doc, "Create feature");
    // Perform related Inventor mutations.
    transaction.End();
}
catch
{
    transaction?.Abort();
    throw;
}
~~~

Do not create one transaction per sketch segment, dimension, or annotation in a batch.

## Update strategy

Build inputs first, mutate in batches, and update at meaningful boundaries.

Avoid:
- Update after every sketch line;
- recalculating physical properties after every feature;
- rebuilding a drawing after every dimension;
- opening and closing the same document repeatedly in one command.

## Save strategy

Low-level geometry helpers should not save documents implicitly. Saving belongs to the command/workflow layer.

When saving:
- validate the path;
- use System.IO.Path and System.IO.File explicitly;
- never silently overwrite user data;
- preserve the distinction between inspect, modify, and modify-and-save commands.
