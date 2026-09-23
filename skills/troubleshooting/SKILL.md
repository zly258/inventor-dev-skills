---
name: troubleshooting
description: Diagnose common Inventor C# Add-In build, load, COM, modeling, drawing, BOM, deployment, and PowerShell failures.
---

# Troubleshooting

Use this skill when the failure does not clearly belong to one API domain.

## Triage order

Check problems in this order:

1. target Inventor/runtime version;
2. x64 architecture;
3. Autodesk Inventor interop reference;
4. .addin manifest path and GUID;
5. managed dependency loading;
6. Add-In load/block state;
7. active document/type/context;
8. units;
9. stale COM geometry/reference;
10. API-specific input geometry;
11. transaction/update state;
12. export/output path.

Do not start by rewriting the algorithm when the Add-In is simply loading the wrong DLL.

## CS0104 ambiguous reference: Path

Typical cause:
- using System.IO;
- using Inventor;
- both expose a type named Path.

Fix:

~~~csharp
string output = System.IO.Path.Combine(folder, fileName);
~~~

or:

~~~csharp
using IOPath = System.IO.Path;
using InventorPath = Inventor.Path;
~~~

For sweep geometry, use Inventor.Path. For filesystem paths, use System.IO.Path.

## File name collision

Use System.IO.File explicitly for filesystem operations.

This also makes code review clearer when Inventor.Document.File or other storage-related API objects are nearby.

## Application collision

WPF code commonly conflicts between:
- Inventor.Application;
- System.Windows.Application.

Alias both. Do not rely on namespace order.

## Transaction collision

If System.Transactions is referenced, Inventor.Transaction can become ambiguous.

Use:

~~~csharp
Inventor.Transaction transaction =
    app.TransactionManager.StartTransaction(doc, "Operation");
~~~

## PowerShell ParserError after a variable

Problem:

~~~powershell
throw "Deployment validation failed for Inventor $version: $required"
~~~

The colon can make the variable reference invalid.

Fix:

~~~powershell
throw "Deployment validation failed for Inventor ${version}: $required"
~~~

Use braces whenever punctuation makes interpolation ambiguous.

## Add-In does not appear

Check:
- .addin file is in a supported directory;
- extension is really .addin, not .addin.txt;
- ClassId matches the AddInServer Guid;
- Assembly path resolves from the manifest location;
- DLL exists;
- x64 build;
- runtime target matches the installed Inventor version;
- required managed dependencies are beside the DLL;
- manifest XML is valid;
- supported-version rules are not excluding the installed Inventor.

## Add-In appears but does not load

Check:
- Add-In Manager status;
- first-load blocking/security rules;
- exception thrown inside Activate;
- FileNotFoundException for a dependency;
- BadImageFormatException caused by architecture mismatch;
- incompatible .NET runtime target;
- wrong Autodesk interop assembly version.

Make Activate lightweight so a nonessential UI/service failure does not hide the real loader problem.

## Ribbon button missing

Check:
- Activate completed;
- ControlDefinition was created;
- InternalName is unique;
- correct Ribbon internal name was used;
- tab/panel was created under the expected environment;
- firstTime logic did not skip required UI;
- UserInterfaceVersion was incremented after a breaking UI layout change.

Do not create duplicate control IDs with translated names.

## Button stops firing after some time

Likely causes:
- event delegate was only held in a local variable;
- ButtonDefinition/event source was garbage collected;
- Deactivate/reload left stale subscriptions.

Keep command definitions and handlers in long-lived Add-In-owned objects.

## COMException / E_FAIL during modeling

Before changing code, inspect:
- active document type;
- profile validity;
- database units;
- operation type;
- body participants;
- stale Face/Edge references;
- feature order;
- self-intersection;
- zero/near-zero geometry.

Log HRESULT and semantic inputs.

## Face/Edge reference worked, then fails after update

Topological indices changed.

Do not persist:
- Faces.Item(3);
- Edges.Item(8).

Use:
- product metadata/AttributeSets;
- ReferenceKey;
- owning feature identity;
- geometric signature fallback.

Rebind after model regeneration.

## Drawing view creation fails

Check:
- target drawing sheet is active where required;
- model Document is valid;
- target Point2d is in sheet coordinates/database units;
- scale is valid;
- representation/model state exists;
- section sketch is valid;
- detail-view parent sheet is active.

## Drawing dimensions are missing

Common causes:
- GeometryIntent is invalid;
- view was not updated before curve extraction;
- wrong curve/endpoint was selected;
- two intents collapse to the same point;
- duplicate-removal logic is too aggressive;
- generated dimension lies outside the sheet;
- code relies only on model features and misses visible geometric relationships.

For stepped shafts and similar parts, derive shoulder stations from the drawing geometry and create segment lengths plus overall length.

## Dimensions are slow

Typical cause:
- candidate pool + repeated creation attempts + view regeneration.

Fix architecture:
- extract once;
- classify once;
- plan all dimensions;
- resolve layout in plain C#;
- create once;
- update once.

## Annotation overlaps

Do not solve every overlap by random retries.

Use deterministic side/quadrant allocation, ordered anchors, spacing levels, then a small bounded local adjustment.

## BOM material/mass cannot be edited

First decide what the cell represents.

Material should come from the model's material definition/asset workflow.

Mass/weight is normally a calculated physical property. A drawing PartsList cell can be a presentation/override layer and is not automatically the writable source of truth.

Do not expose calculated cells as ordinary editable text fields.

## iProperty write works on one file but not another

Check:
- PropertySet identity/localization;
- Model State edit scope;
- member vs factory document;
- property data type;
- read-only/calculated source.

Prefer PropertySet.InternalName/FMTID and ItemByPropId where practical.

## Chinese or non-ASCII file names are corrupted

.NET strings and System.IO APIs are Unicode. Do not manually convert Inventor file paths through ANSI/default code pages.

Avoid:
- ASCIIEncoding for paths;
- byte[] round-trips using Encoding.Default;
- native helper APIs that accept char* when a Unicode variant exists.

Keep the original .NET string from file dialogs/Inventor APIs through System.IO and Inventor calls.

## Export produces no file

Check:
- correct TranslatorAddIn;
- translator SupportsSaveCopyAs;
- source document/type;
- TranslationContext;
- documented option names;
- DataMedium.FileName;
- destination directory exists;
- overwrite policy;
- final file existence/size.

For flat-pattern export, confirm a valid FlatPattern exists first.

## NullReference/CS8602 around COM

Do not suppress nullable warnings globally.

Validate:
- ActiveDocument;
- optional API return values;
- Item lookup results;
- current drawing view;
- referenced document;
- selected entity.

Wrap repeated checks in typed guard helpers.

## Last-resort diagnostic package

When a problem remains, capture:
- Inventor version/build;
- target framework;
- x64/AnyCPU;
- Add-In DLL path;
- .addin contents;
- active document type;
- full exception and HRESULT;
- operation inputs;
- last successful API step.

This usually narrows an "Inventor API is unstable" report to one reproducible condition.
