---
name: addin-lifecycle
description: Implement ApplicationAddInServer correctly, manage Inventor.Application, UI, events, and shutdown.
---

# Add-In Lifecycle

An Inventor Add-In must implement Inventor.ApplicationAddInServer.

## Minimal shape

~~~csharp
using System;
using System.Runtime.InteropServices;
using Inventor;

namespace Product.Addin;

[Guid(AddInGuid)]
public sealed class AddInServer : ApplicationAddInServer
{
    public const string AddInGuid = "00000000-0000-0000-0000-000000000000";

    private Inventor.Application? _application;

    public object? Automation => null;

    public void Activate(ApplicationAddInSite addInSiteObject, bool firstTime)
    {
        _application = addInSiteObject.Application
            ?? throw new InvalidOperationException("Inventor application is unavailable.");

        // Create command definitions.
        // If firstTime, create persistent Ribbon UI.
        // Subscribe required events and keep delegate/source references alive.
    }

    public void Deactivate()
    {
        // Unsubscribe events.
        // Release UI-owned resources.
        _application = null;
    }

    public void ExecuteCommand(int commandID)
    {
        // Legacy command entry point. Most modern Add-Ins use ControlDefinition events.
    }
}
~~~

Replace the sample GUID and use the same value in the .addin ClassId/ClientId unless there is a deliberate reason to separate them.

## Activate

Activate is initialization, not a place for heavy model processing.

Do:
- capture addInSiteObject.Application;
- construct services;
- create ControlDefinitions;
- create persistent UI when firstTime is true;
- subscribe events;
- validate optional dependencies.

Avoid:
- scanning all documents;
- mass model updates;
- long file I/O;
- blocking network calls;
- background access to Inventor COM objects.

## firstTime

Autodesk uses firstTime to distinguish first creation of persistent Inventor UI from later loads.

Typical pattern:
- always ensure ControlDefinitions exist for this process;
- only create Ribbon tabs/panels/controls when firstTime requires it;
- increment UserInterfaceVersion in the manifest when the persisted UI must be rebuilt.

## Deactivate

Always:
- unsubscribe event handlers;
- close owned windows;
- remove temporary state;
- clear strong references to Inventor objects.

Do not depend on process exit to clean up event subscriptions.

## COM lifetime

Do not blanket-call Marshal.FinalReleaseComObject across arbitrary API objects. A shared RCW may still be in use elsewhere. Prefer normal managed lifetime and short local scopes. Explicit release is reserved for carefully controlled interop boundaries with a demonstrated need.

## Event lifetime

A common failure is creating an event source or delegate in a local variable and letting it be collected. Store event sources/delegates on the AddInServer or a long-lived service and unsubscribe during Deactivate.
