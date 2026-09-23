---
name: import-export
description: Import, save, and export Inventor documents with Document APIs, TranslatorAddIn, NameValueMap, DataMedium, STEP, DWG, DXF, PDF, and flat-pattern workflows.
---

# Import and Export

Choose the API based on the operation. Do not treat every output format as a normal SaveAs.

## Native save vs translation

Use native document save APIs for Inventor document ownership/save operations.

Use TranslatorAddIn for publishing/conversion formats such as:
- STEP;
- IGES;
- DWG;
- DXF;
- PDF;
- DWF;
- other installed translators.

Document.SaveAs2 is available in newer Inventor versions. If supporting older releases, hide save-version differences behind a compatibility service.

## Translator workflow

The common TranslatorAddIn pattern is:

1. locate the translator Add-In;
2. create TranslationContext;
3. create NameValueMap;
4. create DataMedium;
5. call HasSaveCopyAsOptions when options are supported;
6. set only documented options;
7. set target filename;
8. call SaveCopyAs.

Conceptually:

~~~csharp
TranslatorAddIn translator = ResolveTranslator(app, translatorId);

TranslationContext context =
    app.TransientObjects.CreateTranslationContext();

NameValueMap options =
    app.TransientObjects.CreateNameValueMap();

DataMedium medium =
    app.TransientObjects.CreateDataMedium();

medium.FileName = outputPath;

if (translator.HasSaveCopyAsOptions(source, context, options))
{
    // Set documented translator options.
}

translator.SaveCopyAs(source, context, options, medium);
~~~

Keep translator IDs and option names in one central catalog. Do not scatter GUID strings across commands.

## STEP

For STEP export:
- confirm the source document type is supported;
- choose protocol/options deliberately;
- write to a validated path;
- verify the file exists after export;
- do not overwrite unless explicitly allowed.

If the workflow prefers STEP as the neutral delivery format, make that a product policy in the export service rather than embedding it in geometry code.

## Drawing PDF

PDF is a translator/publishing operation.

Before exporting:
- ensure drawing views are up to date;
- ensure all sheets intended for output are included;
- apply documented PDF options;
- verify output path and page/sheet scope.

Do not export while automatic drawing generation is still mutating the sheet.

## DWG/DXF

DWG/DXF translator options can be extensive and version-sensitive.

Prefer:
- a checked-in configuration/INI file when the Autodesk translator supports it;
- a typed application configuration that generates NameValueMap values;
- one tested export path.

Do not guess option names.

## Sheet-metal flat pattern

FlatPattern can be supplied to supported translators. Inventor also supports the flat-pattern DataIO workflow for manufacturing DXF/DWG output.

Use a separate flat-pattern export configuration for:
- outer profile layer;
- inner profile layer;
- bend up/down layers;
- tangent lines;
- arc centers;
- spline simplification;
- machine-specific requirements.

## Import/open

When opening non-native data:
- use the appropriate TranslatorAddIn if import options are needed;
- validate translator availability;
- configure TranslationContext;
- treat the resulting Inventor document as a new model that still needs validation.

Do not assume imported BRep has the same feature semantics as native Inventor modeling.

## File-system safety

Always use explicit System.IO names in Inventor code:

~~~csharp
string directory = System.IO.Path.GetDirectoryName(outputPath)!;

if (!System.IO.Directory.Exists(directory))
{
    System.IO.Directory.CreateDirectory(directory);
}

if (System.IO.File.Exists(outputPath) && !allowOverwrite)
{
    throw new IOException("Output file already exists.");
}
~~~

This avoids Inventor.Path / Inventor.File namespace conflicts.

## Batch export

For multiple files:
1. group by document/output type;
2. resolve translator once;
3. reuse stable option definitions;
4. open each document once;
5. export all requested formats;
6. close only documents the command opened;
7. produce a result per file.

Never close a document that was already open by the user unless the workflow explicitly owns it.

## Validation

A successful COM call is not enough.

Validate:
- target file exists;
- target size is non-zero;
- expected number of drawing sheets/files were produced;
- source document did not unexpectedly become dirty;
- errors include source, destination, translator, options, and HRESULT.
