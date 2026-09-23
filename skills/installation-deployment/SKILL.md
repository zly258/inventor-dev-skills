---
name: installation-deployment
description: Configure .addin manifests and install/uninstall Inventor Add-Ins safely for per-user or all-user deployment.
---

# Installation and Deployment

Prefer Autodesk's registry-free .addin mechanism.

## Manifest

Example:

~~~xml
<Addin Type="Standard">
  <ClassId>{YOUR-GUID}</ClassId>
  <ClientId>{YOUR-GUID}</ClientId>
  <DisplayName>Product</DisplayName>
  <Description>Product Inventor Add-In</Description>
  <Assembly>Product.Addin.dll</Assembly>
  <OSType>Win64</OSType>
  <LoadAutomatically>1</LoadAutomatically>
  <UserUnloadable>1</UserUnloadable>
  <Hidden>0</Hidden>
  <DataVersion>1</DataVersion>
  <LoadBehavior>0</LoadBehavior>
  <UserInterfaceVersion>1</UserInterfaceVersion>
</Addin>
~~~

Rules:
- ClassId must match the Guid attribute on the Add-In server class.
- Keep ClientId stable for a product because Inventor associates UI and other owned objects with it.
- Prefer a relative Assembly path and keep the DLL beside the manifest.
- Use Win64.
- Change UserInterfaceVersion when you intentionally need Inventor to rebuild persistent UI.

## Supported locations

Registry-free manifests can be placed in Autodesk-supported Add-In locations, including:

- all users, version independent:
  %ALLUSERSPROFILE%\Autodesk\Inventor Addins\
- all users, version dependent for Inventor 2024+:
  %PROGRAMFILES%\Autodesk\Inventor 20xx\Bin\Addins\
- per user, version dependent:
  %APPDATA%\Autodesk\Inventor 20xx\Addins\
- per user, version independent:
  %APPDATA%\Autodesk\ApplicationPlugins

For development and simple deployment, a per-user version-dependent folder is easy to reason about:

~~~text
%APPDATA%\Autodesk\Inventor 2026\Addins\Product\
  Product.addin
  Product.Addin.dll
  Product.Core.dll
~~~

## Install script pattern

~~~powershell
param(
    [string]$InventorVersion = "2026"
)

$target = Join-Path $env:APPDATA "Autodesk\Inventor $InventorVersion\Addins\Product"
New-Item -ItemType Directory -Force -Path $target | Out-Null

Copy-Item ".\bin\*" $target -Recurse -Force
Write-Host "Installed to $target"
~~~

## PowerShell interpolation trap

This is invalid or ambiguous when a colon immediately follows a variable:

~~~powershell
throw "Deployment validation failed for Inventor $version: $required"
~~~

Use braces:

~~~powershell
throw "Deployment validation failed for Inventor ${version}: $required"
~~~

This exact pattern is easy to miss in install.ps1 validation messages.

## Automatic loading and blocking

Inventor can block an Add-In on first load according to its Add-In load rules. Do not work around this with random registry edits.

For enterprise deployment:
- package the manifest and DLLs consistently;
- keep GUID and assembly path stable;
- use an IT-controlled installer/script;
- handle AddInLoadRules.xml only as part of an intentional administrator deployment policy;
- test clean-machine first launch.

## Validation checklist

After install:
1. manifest exists in a scanned Add-In location;
2. Assembly resolves relative to the manifest;
3. DLL and dependencies exist;
4. architecture is x64;
5. GUID matches;
6. target runtime matches Inventor;
7. Add-In appears in Add-In Manager;
8. loading does not immediately throw;
9. Ribbon UI is present after restart if expected.

## Uninstall

Remove only the product's own folder/files. Do not delete the parent Autodesk Addins directory.
