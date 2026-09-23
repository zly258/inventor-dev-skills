---
name: bom-properties
description: Work with Inventor assembly BOMs, drawing PartsLists, iProperties, material, mass, item numbers, and model-state property scope.
---

# BOM, PartsList, and iProperties

Keep three concepts separate:

1. model/document properties;
2. assembly BOM data;
3. drawing PartsList presentation/overrides.

They are related but not interchangeable.

## Assembly BOM

Start from:

~~~csharp
AssemblyComponentDefinition definition = assembly.ComponentDefinition;
BOM bom = definition.BOM;
~~~

Enable the required BOM view deliberately, such as structured or parts-only, before reading it.

When locating BOM views, prefer stable enum/type information where available instead of relying only on localized display text such as "Structured" or "Parts Only".

## Drawing PartsList

PartsList belongs to a drawing sheet and represents BOM information in the drawing.

It can:
- sort;
- renumber;
- export;
- contain display overrides;
- save supported item overrides back to the BOM.

Do not assume editing a PartsList cell changes the underlying model property.

PartsList.SaveItemOverridesToBOM exists specifically for supported item overrides. Treat other displayed/calculated cells according to their actual source.

## iProperties

Documents expose PropertySets.

Built-in PropertySet display names can be localized. Prefer:
- PropertySet.InternalName/FMTID where practical;
- ItemByPropId for known built-in property IDs;
- a small centralized mapping layer.

Avoid scattering strings like "Design Tracking Properties" throughout business code.

For custom properties, use the user-defined/custom PropertySet and a product-specific naming convention.

## Example property helper

Conceptually:

~~~csharp
PropertySet FindPropertySet(Document doc, string internalName)
{
    foreach (PropertySet set in doc.PropertySets)
    {
        if (string.Equals(set.InternalName, internalName, StringComparison.OrdinalIgnoreCase))
            return set;
    }

    throw new InvalidOperationException("Required property set was not found.");
}
~~~

Centralize locale-sensitive fallbacks if they are required for old documents.

## Material

Material is model design data. Do not implement "change material" by writing text into a drawing PartsList cell.

Use the model's material/asset API appropriate to the target Inventor version, then update the document and physical properties.

A text property named "Material" may be useful for reporting, but it should not become a second independent source of truth.

## Mass / weight

Mass is normally derived from:
- geometry;
- material density;
- physical properties;
- model state.

A BOM or PartsList mass/weight cell can therefore be calculated and not freely editable as a normal string field.

If the product intentionally supports a mass override, implement that as an explicit physical-property override workflow. Do not silently replace calculated mass with a static PartsList value.

## Model states

iProperties can vary by Model State. Before reading/writing properties, know:
- active ModelStateName;
- whether edits should apply to the member or factory;
- whether the document represents a member/factory document.

A property update that appears "random" is often a scope problem rather than a failed assignment.

## Item numbers

Treat item number as BOM identity, not just text.

When changing item numbers:
- update the correct BOM view/row;
- consider merged rows;
- preserve balloon/PartsList consistency;
- validate drawing references afterward.

## Quantities

Quantity can be affected by:
- BOM structure;
- occurrence suppression/state;
- virtual components;
- merged rows;
- model states/iAssembly;
- custom quantity settings.

Do not calculate final BOM quantity by simply counting ComponentOccurrence objects.

## Custom properties

For Add-In-owned metadata:
- prefer a dedicated custom property name prefix or AttributeSet;
- use PropertySet.SetPropertyValues for batch updates when appropriate;
- write data types consistently;
- do not change unrelated user-defined properties.

## Read-only/calculated fields

Before exposing a grid cell as editable, classify the field:
- writable document property;
- model-derived property;
- BOM-owned value;
- drawing-only override;
- calculated/read-only value.

This avoids the common UX bug where the editor accepts a value that Inventor immediately recalculates or ignores.

## Batch performance

For many documents:
1. read required properties in one pass;
2. compute translations/changes in plain C#;
3. write only changed values;
4. update/save once per document as required.

Do not repeatedly open the same document for each BOM column.
