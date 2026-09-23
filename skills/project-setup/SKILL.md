---
name: project-setup
description: Configure C# Inventor Add-In projects, framework targets, x64, references, WPF, and build output.
---

# Project Setup

## Target framework

For Inventor 2025+:
- prefer net8.0-windows;
- use Visual Studio 2022 17.8 or later;
- start from the Autodesk Inventor Add-In template when available.

For older Inventor releases:
- use the framework target generated/recommended by that release's SDK;
- net48 is a practical legacy baseline only when compatible with the target Inventor version.

Do not try to produce one binary that spans incompatible runtime generations. Prefer separate target-specific builds behind shared source projects/libraries.

## Platform

Use x64.

Recommended properties:

~~~xml
<PropertyGroup>
  <TargetFramework>net8.0-windows</TargetFramework>
  <PlatformTarget>x64</PlatformTarget>
  <Platforms>x64</Platforms>
  <Nullable>enable</Nullable>
  <ImplicitUsings>enable</ImplicitUsings>
  <UseWPF>true</UseWPF>
</PropertyGroup>
~~~

For a legacy project:

~~~xml
<PropertyGroup>
  <TargetFrameworkVersion>v4.8</TargetFrameworkVersion>
  <PlatformTarget>x64</PlatformTarget>
</PropertyGroup>
~~~

Only enable UseWPF when the Add-In actually owns WPF UI.

## Inventor interop reference

Use the Inventor interop assembly supplied by the installed Autodesk Inventor/SDK for the target release. Avoid copying a random Autodesk.Inventor.Interop.dll from another machine or release.

Typical rule:
- Inventor interop reference: do not package a conflicting private copy unless the Autodesk template for that version explicitly requires it.
- Your own managed dependencies: copy them next to the Add-In DLL.
- Keep Autodesk API reference versioning centralized in the project or Directory.Build.props.

## Suggested solution layout

~~~text
src/
  Product.Addin/
    AddInServer.cs
    Commands/
    UI/
    Services/
  Product.Core/
    Geometry/
    Drawing/
    Models/
tests/
build.ps1
install.ps1
uninstall.ps1
Product.addin
~~~

Keep Inventor COM types mostly inside Product.Addin or a dedicated Inventor adapter layer. Core algorithms should accept plain C# models whenever possible.

## Build output

Prefer one predictable output folder for the deployable Add-In package:

~~~text
bin/
  Product.Addin.dll
  Product.addin
  Product.Core.dll
  dependency.dll
~~~

Do not generate unnecessary nested framework/configuration directories in the final deployment package unless multiple targets are intentionally shipped.

## WPF and Inventor

If WPF is used:
- Inventor.Application and System.Windows.Application are ambiguous: alias them.
- WPF Point and Inventor.Point are different types: alias or fully qualify.
- keep WPF view models free of Inventor COM objects where possible;
- invoke Inventor API calls back on the Inventor/UI thread.

## Debugging

Configure Visual Studio to start the matching Inventor.exe and deploy the current DLL/.addin to a supported Add-In directory before launch.

Never debug against an old copied DLL while Visual Studio builds a different output location.
