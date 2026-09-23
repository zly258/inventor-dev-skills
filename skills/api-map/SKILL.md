---
name: api-map
description: Quick object-model map for Autodesk Inventor C# development and common entry points by task.
---

# Inventor API Map

Use this as a navigation cheat sheet, not as a replacement for the focused skills.

## Application

~~~text
Inventor.Application
  ActiveDocument
  Documents
  CommandManager
    ControlDefinitions
  UserInterfaceManager
    Ribbons
    Environments
    UserInterfaceEvents
  ApplicationAddIns
  TransactionManager
  TransientGeometry
  TransientObjects
  UnitsOfMeasure
~~~

Important:
- Inventor.Application is not System.Windows.Application.
- Application.TransientGeometry creates Point, Point2d, Vector, Matrix, and similar transient geometry.
- Application.TransientObjects creates ObjectCollection, NameValueMap, TranslationContext, DataMedium, and other general transient objects.

## Document

~~~text
Document
  DocumentType
  FullFileName
  Dirty
  PropertySets
  FilePropertySets
  UnitsOfMeasure
  ReferenceKeyManager
  DocumentEvents
~~~

Derived document types:
- PartDocument;
- AssemblyDocument;
- DrawingDocument;
- PresentationDocument.

Check DocumentType or the runtime type before casting.

Document.File is not System.IO.File.

## Part

~~~text
PartDocument
  ComponentDefinition
    Sketches
    Sketches3D
    WorkPlanes
    WorkAxes
    WorkPoints
    UserCoordinateSystems
    Features
      ExtrudeFeatures
      RevolveFeatures
      SweepFeatures
      LoftFeatures
      HoleFeatures
      FilletFeatures
      ChamferFeatures
      ShellFeatures
      RectangularPatternFeatures
      CircularPatternFeatures
      MirrorFeatures
    SurfaceBodies
    Parameters
    ReferenceComponents
~~~

Sheet metal uses SheetMetalComponentDefinition, which derives from PartComponentDefinition and adds sheet-metal-specific APIs and FlatPattern.

## Sketch

~~~text
PlanarSketch
  SketchLines
  SketchCircles
  SketchArcs
  SketchSplines
  GeometricConstraints
  DimensionConstraints
  Profiles
~~~

Create a sketch through PartComponentDefinition.Sketches.

Create solid profiles through Profiles.AddForSolid.

## Assembly

~~~text
AssemblyDocument
  ComponentDefinition
    Occurrences
      ComponentOccurrence
        Definition
        Transformation
        CreateGeometryProxy
    Constraints
    BOM
    RepresentationsManager
~~~

Use geometry proxies for assembly-context geometry.

Do not count occurrences as a substitute for reading the BOM.

## Drawing

~~~text
DrawingDocument
  Sheets
    Sheet
      DrawingViews
      DrawingDimensions
        GeneralDimensions
      DrawingNotes
      GeneralNotes
      LeaderNotes
      HoleThreadNotes
      Centermarks
      Centerlines
      Balloons
      PartsLists
      HoleTables
~~~

Exact collection access varies by annotation family; start from the Sheet and inspect the target API in the Autodesk documentation.

## Drawing views

Core methods:
- DrawingViews.AddBaseView
- DrawingViews.AddProjectedView
- DrawingViews.AddSectionView / AddSectionView2
- DrawingViews.AddDetailView
- DrawingViews.AddAuxiliaryView

Use DrawingView.DrawingCurves to map model geometry into drawing geometry.

## Drawing dimensions

Core path:

~~~text
Sheet
  CreateGeometryIntent(...)
  DrawingDimensions
    GeneralDimensions
      AddLinear
      AddLinear2
      AddAngular
      AddDiameter
      AddRadius
~~~

Placement points are sheet Point2d values.

## BOM and properties

~~~text
AssemblyComponentDefinition
  BOM
    BOMViews
      BOMView
        BOMRows

Document
  PropertySets
    PropertySet
      Property
      ItemByPropId
~~~

Drawing PartsList is a presentation of BOM data, not the same object as Assembly BOM.

## Ribbon UI

~~~text
Application.CommandManager
  ControlDefinitions
    AddButtonDefinition

Application.UserInterfaceManager
  Ribbons
    Ribbon
      RibbonTabs
        RibbonTab
          RibbonPanels
            RibbonPanel
              CommandControls
~~~

Stable InternalName and ClientId values are more important than user-facing translated captions.

## Import/export

~~~text
Application.ApplicationAddIns
  TranslatorAddIn

Application.TransientObjects
  TranslationContext
  NameValueMap
  DataMedium
~~~

TranslatorAddIn.SaveCopyAs is the standard path for many neutral/publishing formats.

## Common object-name collisions

| Meaning | Explicit type |
| --- | --- |
| file path helper | System.IO.Path |
| filesystem file | System.IO.File |
| Inventor sweep path | Inventor.Path |
| Inventor app | Inventor.Application |
| WPF app | System.Windows.Application |
| Inventor 3D point | Inventor.Point |
| Inventor 2D point | Inventor.Point2d |
| WPF point | System.Windows.Point |
| Inventor transaction | Inventor.Transaction |
| .NET environment | System.Environment |
| Inventor UI environment | Inventor.Environment |

When in doubt, fully qualify the type at the call site first; introduce a clear using alias after the code is correct.
