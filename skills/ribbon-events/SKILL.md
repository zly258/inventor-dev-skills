---
name: ribbon-events
description: Create Inventor Ribbon commands and manage command/document/application events with correct Add-In lifetime.
---

# Ribbon UI and Events

## Command definitions

Create commands through Application.CommandManager.ControlDefinitions.

For buttons, use ControlDefinitions.AddButtonDefinition.

Important fields:
- DisplayName: localized user-facing text;
- InternalName: globally unique stable internal ID;
- CommandTypesEnum classification;
- ClientId: your Add-In GUID;
- description/tooltip;
- icons;
- ButtonDisplayEnum.

InternalName is identity. Do not change it casually between releases.

## Ribbon structure

UI customization starts from:

~~~csharp
UserInterfaceManager ui = app.UserInterfaceManager;
Ribbons ribbons = ui.Ribbons;
~~~

Then resolve the intended ribbon, tab, panel, and add a CommandControl referencing your ControlDefinition.

Conceptual structure:

~~~text
Application
  UserInterfaceManager
    Ribbons
      Ribbon
        RibbonTabs
          RibbonTab
            RibbonPanels
              RibbonPanel
                CommandControls
~~~

Use Inventor's actual internal ribbon/tab IDs when extending built-in UI. Do not identify built-in UI only by localized captions.

## firstTime rule

Persistent Ribbon UI is normally created when Activate receives firstTime = true.

ControlDefinitions/events needed for the current process should still be initialized on every activation.

When you intentionally change persistent UI structure, increment UserInterfaceVersion in the .addin manifest so Inventor can rebuild the Add-In-owned UI.

## Button event lifetime

Keep the ButtonDefinition and its event subscription in a long-lived object.

Bad pattern:
- create ButtonDefinition in a local helper;
- subscribe a temporary delegate;
- lose all managed references.

Better:
- AddInServer owns a CommandRegistry;
- CommandRegistry owns ButtonDefinition instances and handlers;
- Deactivate unsubscribes handlers.

## Command architecture

A Ribbon handler should be thin:

~~~text
Button OnExecute
  -> validate active document/context
  -> call application service/command
  -> show concise user result/error
~~~

Do not put hundreds of lines of Inventor geometry logic directly in the click event.

## Events

Useful event areas include:
- ApplicationEvents;
- DocumentEvents;
- UserInterfaceEvents;
- PartEvents / AssemblyEvents where appropriate;
- command/interaction events.

Only subscribe to events the Add-In needs.

## Event recursion

Automation can trigger events that trigger automation again.

Use a scoped guard:

~~~csharp
if (_isUpdating)
    return;

try
{
    _isUpdating = true;
    // controlled mutation
}
finally
{
    _isUpdating = false;
}
~~~

For complex cases use a counter/scoped disposable guard rather than one global Boolean.

## Event performance

Never do heavy work synchronously for every low-level change event.

Instead:
- filter by document/type/action;
- coalesce repeated changes;
- invalidate cached analysis;
- recompute only when the user invokes the relevant command or at a safe high-level event.

## WPF windows

If the Add-In opens WPF windows:
- keep UI state separate from Inventor COM state;
- do not bind COM objects directly into long-lived ViewModels;
- marshal Inventor mutations back to the Inventor/UI thread;
- close/dispose Add-In-owned windows during Deactivate.

Use aliases for Inventor.Application vs System.Windows.Application.

## Localization

Separate:
- stable internal command IDs;
- localized DisplayName;
- localized tooltip/description.

Do not use translated UI text as an API lookup key.

## Icons

Keep icon creation behind one helper. Do not duplicate image-to-IPicture conversion code in every command.

If a command does not benefit from an icon, do not force decorative icons into the UI.

## Failure behavior

A command handler should:
- catch expected validation errors and explain them;
- log COMException HRESULT/details;
- leave document state consistent;
- abort its transaction on failure;
- never use empty catch blocks.
