---
name: assembly
description: Create, place, traverse, constrain, transform, and inspect Inventor assembly occurrences safely.
---

# Assembly Development

## Entry point

~~~csharp
if (app.ActiveDocument is not AssemblyDocument assembly)
{
    throw new InvalidOperationException("An assembly document must be active.");
}

AssemblyComponentDefinition definition = assembly.ComponentDefinition;
ComponentOccurrences occurrences = definition.Occurrences;
~~~

## Place occurrences

ComponentOccurrences is the main collection for placed components.

Typical inputs:
- source part/assembly path;
- placement Matrix;
- optional representation/model-state options on APIs that support them.

Always validate a source path with System.IO.File and System.IO.Path. Do not confuse filesystem Path with Inventor.Path.

Keep placement transforms explicit. A default identity transform is appropriate only when intended.

## Transform matrices

Use Application.TransientGeometry.CreateMatrix.

Keep a clear convention for:
- source coordinate system;
- assembly coordinate system;
- translation units;
- rotation units/order.

Do not accumulate arbitrary incremental transforms when a deterministic absolute transform can be computed.

## Grounding

Grounding and constraints are different concepts.

Use Grounded only when the design intent is to fix an occurrence. Do not ground every occurrence just to suppress motion.

For bulk top-level grounding, newer Inventor versions also expose assembly-level occurrence property operations. Keep version-specific usage behind a compatibility helper.

## Assembly constraints

Common constraint families include:
- mate;
- flush;
- insert;
- angle;
- tangent;
- symmetry and other supported relationships.

Resolve geometry in assembly context. Geometry from a native part definition often needs a proxy before it can be used reliably in assembly-level operations.

## Geometry proxies

ComponentOccurrence.CreateGeometryProxy converts native component geometry into assembly-context proxy geometry.

Use proxies when:
- constraining geometry from occurrences;
- comparing positions in assembly space;
- using occurrence-owned faces/edges with assembly APIs.

Do not mix native part-space geometry and assembly-space proxy geometry in the same calculation without an explicit transform.

## Traversal

Assembly structures are recursive.

When walking occurrences:
1. handle suppressed/unavailable occurrences;
2. distinguish part vs subassembly;
3. preserve the occurrence path/context;
4. avoid reopening the same referenced document repeatedly;
5. detect repeated references and virtual components where relevant.

Return plain DTO records to non-COM code instead of exposing live ComponentOccurrence objects throughout the application.

## Interference

AssemblyComponentDefinition.AnalyzeInterference can evaluate interference between one collection or two occurrence collections.

Treat interference analysis as an explicit expensive operation. Do not run it automatically after every minor edit.

## Model states and representations

Do not assume the document's current representation is the only valid state.

When automation depends on:
- model state;
- design view representation;
- positional representation;
- iAssembly member;

capture and validate the active context.

## Replacement and references

When replacing components:
- preserve intended constraints when possible;
- validate occurrence identity after replacement;
- do not identify occurrences only by Item(index);
- prefer occurrence name/path plus document identity or product-owned metadata.

## BOM impact

Occurrence BOM structure and assembly BOM settings affect PartsList and balloon behavior. Modeling code that changes occurrence structure should not silently assume the drawing BOM will remain unchanged.

Load the bom-properties skill whenever assembly changes affect downstream documentation.
