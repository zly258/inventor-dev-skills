---
name: com-interop-conflicts
description: Avoid C# type collisions and COM lifetime/threading failures when using Inventor, WPF, System.IO, and System.Transactions.
---

# COM Interop and Name Conflicts

Many Inventor failures are not geometry problems; they are namespace, RCW, thread, or object-lifetime problems.

## Common ambiguous types

### Path

Inventor has Inventor.Path for sweep/path geometry. System.IO has System.IO.Path for file paths.

Prefer:

~~~csharp
var folder = System.IO.Path.GetDirectoryName(fileName);
var output = System.IO.Path.Combine(folder!, "output.idw");
~~~

or:

~~~csharp
using IOPath = System.IO.Path;
using InventorPath = Inventor.Path;
~~~

Never write Path.Combine in a file that also imports Inventor unless the meaning is unambiguous.

### File

Use System.IO.File explicitly for filesystem operations:

~~~csharp
if (!System.IO.File.Exists(modelPath))
{
    throw new FileNotFoundException("Model was not found.", modelPath);
}
~~~

Do not assume File resolves to System.IO.File when Inventor types are imported.

### Application

WPF and Inventor both commonly introduce Application into scope.

~~~csharp
using InventorApplication = Inventor.Application;
using WpfApplication = System.Windows.Application;
~~~

### Point / Vector / Color

Potential collisions include:
- Inventor.Point / Point2d
- System.Windows.Point
- System.Drawing.Point
- Inventor.Color
- System.Drawing.Color
- System.Windows.Media.Color

Alias by semantic role instead of relying on namespace order.

### Transaction

Inventor.Transaction and System.Transactions.Transaction are unrelated. In Inventor mutation code, explicitly use Inventor.Transaction.

### Environment

Inventor.Environment and System.Environment are unrelated. Use System.Environment for process/OS data.

## COMException handling

Do not do this:

~~~csharp
try
{
    // COM work
}
catch
{
}
~~~

Use narrow catches and preserve the HRESULT:

~~~csharp
catch (System.Runtime.InteropServices.COMException ex)
{
    logger.Error($"Inventor COM failure 0x{ex.HResult:X8}: {ex.Message}");
    throw;
}
~~~

If a known HRESULT is intentionally handled, document why.

## Thread affinity

Inventor's live API objects belong to Inventor's in-process COM/UI context.

Bad:

~~~csharp
await Task.Run(() => partDoc.ComponentDefinition.Features.Count);
~~~

Better:
1. read required Inventor state on the Inventor thread;
2. copy values into plain DTOs;
3. perform pure computation in the background if worthwhile;
4. return to the Inventor thread to apply mutations.

## RCW lifetime

Avoid aggressive Marshal.ReleaseComObject/FinalReleaseComObject on normal Inventor objects. Releasing an RCW that other code still references can create difficult intermittent failures.

Prefer:
- short-lived local variables;
- explicit event unsubscribe;
- no global caches of fine-grained geometry objects;
- stable IDs/reference keys instead of holding faces/edges forever.

## Nullability

COM APIs can return null or invalid objects in edge cases even when older sample code assumes otherwise. Modern C# code should validate:
- ActiveDocument;
- SelectSet contents;
- referenced document availability;
- DrawingCurve/GeometryIntent lookup;
- PropertySet/Property existence.

## Dynamic

Avoid dynamic for core Inventor code. It removes compile-time checks exactly where overloads, optional parameters, enums, and COM signatures are already error-prone.
