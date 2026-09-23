---
name: performance-reliability
description: Make Inventor C# Add-Ins fast and reliable by reducing COM round-trips, batching updates, isolating computation, and preserving document consistency.
---

# Performance and Reliability

Inventor Add-Ins run in-process and can be fast, but COM-style object access becomes expensive when code performs thousands of tiny calls.

## Measure first

Instrument major phases:

~~~text
validate
extract
analyze
layout
mutate
update
export
total
~~~

A command that feels "slow" should report which phase is slow.

## Minimize COM round-trips

Bad pattern:

~~~text
for every candidate
  query view
  query curve
  query model entity
  query properties
  try create
  update
  retry
~~~

Better:

~~~text
COM extraction once
  -> plain DTO list
pure C# analysis/layout
  -> final operation plan
COM mutation once
  -> one controlled update
~~~

This pattern is especially important for automatic drawing dimensions and annotations.

## Cache carefully

Good short-lived caches:
- active document/component definition;
- one view's extracted curve metadata;
- one command's PropertySet handles;
- translator Add-In resolution.

Bad long-lived caches:
- arbitrary Face/Edge COM objects across major model edits;
- ComponentOccurrence references after replacement;
- DrawingCurve references after view regeneration.

For long-lived identity, store stable metadata/reference keys and rebind.

## Update boundaries

Do not call document/view update after every mutation.

Batch:
- sketch geometry;
- dimensions;
- notes;
- property changes.

Then update at a meaningful boundary and validate.

## Transactions

Use one transaction for one logical operation.

Benefits:
- coherent undo;
- rollback on failure;
- fewer partially-applied states.

Do not nest transactions casually and do not leave one open across UI waits or unrelated work.

## Background computation

Inventor API calls stay on the Inventor/UI thread.

Safe background work can include:
- sorting DTOs;
- graph/layout algorithms;
- text processing;
- numeric optimization;
- file-independent business rules.

Unsafe background work includes reading or mutating live Inventor COM objects.

## Geometry algorithms

Avoid O(n^2) comparisons when geometry lists grow.

Use:
- axis buckets;
- tolerance-based coordinate clustering;
- spatial bins;
- dictionaries keyed by normalized geometry signature;
- precomputed bounding boxes.

For dimensioning, classify once by orientation/source and only compare entities within relevant groups.

## Error boundaries

Catch exceptions at meaningful command/service boundaries.

Inside low-level helpers:
- add context;
- rethrow;
- do not convert every failure to false/null.

At the command boundary:
- abort transaction;
- log details;
- report a concise user message;
- preserve diagnostics.

## COM exception diagnostics

Always capture:
- HRESULT;
- API operation;
- document path/name;
- entity/feature identity;
- important input values.

Do not log only ex.Message.

## Partial success

For batch operations, return a structured result:

~~~text
Succeeded
Failed
Skipped
Warnings
Duration
PerItemResults
~~~

Do not let one optional annotation hide the success/failure state of the entire drawing.

## Document ownership

Track whether a document was:
- already open by the user;
- opened by the command;
- newly created by the command.

Only close documents owned by the command.

## Idempotency

Generation commands should be safe to rerun.

Use AttributeSets/stable IDs to distinguish:
- generated objects;
- user-created objects;
- obsolete generated objects.

Rerun should update/replace its own artifacts, not duplicate them.

## Empty catch blocks

Empty catch blocks are prohibited in Inventor integration code.

If an API failure is expected:
- catch the specific exception;
- document the accepted HRESULT/condition;
- record a diagnostic when useful.

Silent failure makes COM automation appear random and is one of the hardest classes of bugs to maintain.

## Reliability checklist

Before calling a feature "done":
- works on a clean Inventor launch;
- works when no document is active;
- rejects wrong document type cleanly;
- handles unsaved documents;
- handles localized UI/property names;
- handles missing dependencies;
- handles rerun;
- aborts transaction on failure;
- leaves no orphan UI/event subscriptions;
- performs acceptably on large models/drawings.
