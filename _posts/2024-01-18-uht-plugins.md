---
tags: 
  - unreal
title: "Extend the Unreal Header Tool (UHT) with Plugins"
categories:
  - Engine
---
<img src="https://img.shields.io/badge/Unreal%20Engine-5.3-informational" alt="Written for Unreal Engine 5.3"> <img src="https://img.shields.io/badge/-C%2B%2B-orange" alt="C++"> <img src="https://img.shields.io/badge/-C%23-green" alt="C#">

## About the Unreal Header Tool
The Unreal Header Tool (UHT) parses and generates code for `UObject` types. Many of the C++ macros in the Unreal Engine source code are actually implemented by the UHT.

For example, given a simple `UObject`:

```cpp
// File: Example.h

#pragma once

#include "Example.generated.h"

/** This is an example UObject. */
UCLASS()
class UExample : public UObject
{
    GENERATED_BODY()
};
```

The UHT generates [two files named `Example.generated.h` and `Example.gen.cpp`](https://gist.github.com/the-unrealist/0aa6b16d1a89c13cd0065b685b9a0bce) with code generated for the `UCLASS()` and `GENERATED_BODY()` macros.

This is how Unreal implements [reflection](https://docs.unrealengine.com/5.3/en-US/unreal-object-handling-in-unreal-engine/#run-timetypeinformationandcasting) and [garbage collection](https://docs.unrealengine.com/5.3/en-US/unreal-object-handling-in-unreal-engine/#garbagecollection) in a language that does not support these. Serialization, network replication, and editor integration are also implemented as generated code.

## Exporters
C++ code generation is implemented as an **exporter** in the UHT.

An exporter processes the parsed code and then optionally *exports* files to the `Intermediate` directory. An exporter may serve as an analyzer and not export any files—as demonstrated in the `Stats` exporter sample.

UHT implements these exporters under `/Engine/Source/Programs/Shared/EpicGames.UHT/Exporters/`.

|Name|Enabled by default?|Purpose|
|----|-------------------|-------|
|CodeGen|Yes|Generates C++ code for the `UObject` system.|
|Json|No|A sample exporter that shows how to output files. This exporter dumps all UObjects into a JSON file for each package.|
|Stats|No|A sample exporter that shows how to log details about types in the codebase.|

Exporters that are not enabled by default are triggered by adding `-<EXPORTER_NAME>` to the commandline arguments for the UHT.

The following command will execute both the `Stats` and `Json` exporters. Replace `MyGameEditor`, `Win64`, and `Development` with the desired target, platform, and build configuration. 

```shell
C:\UE_5.3\Engine\Build\BatchFiles>RunUBT.bat -Mode=UnrealHeaderTool -Stats -Json "-Target=MyGameEditor Win64 Development -Project=\"C:/Path/To/MyGame.uproject\""
```

## Extend the UHT with plugins
The Unreal Build Tool (UBT) scans for [extension plugins](https://docs.unrealengine.com/5.3/en-US/unreal-header-tool-for-unreal-engine/#extendinguhtwithscriptgenerators) in both engine and your game's source, and automatically activates them.

At this time, only exporters are supported in UBT plugins.

I created a UBT plugin called [**Specifier Reference Viewer**](https://github.com/the-unrealist/specifier-reference-viewer) that scans for all specifier keywords used in the source code for the engine, game, and all plugins. I recommend looking at my plugin's source code to learn more about creating UBT plugins.

Let's walk through the steps needed to create an exporter plugin.

### 1. Create a plugin
Create a `.uplugin` with at least one module. A module is required even if you're not outputting any file, and it must have the `Runtime` type.

```json
{
    "EnabledByDefault": false,
    "Modules": [
        {
            "Name": "MyCustomExporter",
            "Type": "Runtime",
            "LoadingPhase": "PreDefault"
        }
    ]
}
```

### 2. Enable the plugin in the editor
In your game's `.uproject`, enable the plugin only for the `Editor` target.

```json
{
    "Plugins": [
        {
            "Name": "MyCustomExporterPlugin",
            "Enabled": true,
            "TargetAllowList": [
                "Editor"
            ]
        }
    ]
}
```

### 3. Add code to the module
A module must have at least one `UObject` in it to be added to the `.uhtmanifest` file. An empty module will not be added to the `.uhtmanifest` file and the custom exporter will not execute.

### 4. Create the C# project
Create a C# project file named `PluginName.ubtplugin.csproj`. It must have the `.ubtplugin.csproj` extension to be detected by UHT.

This file should be configured to:
* import `/Engine/Source/Programs/Shared/UnrealEngine.csproj.props`,
* reference the `EpicGames.Build`, `EpicGames.Core`, `EpicGames.UHT`, and `UnrealBuildTool` assemblies, and
* output the compiled binaries to `<PROJECT_NAME>/Binaries/DotNET/UnrealBuildTool/Plugins/<PLUGIN_NAME>`.

Instead of creating the project file from scratch, it may be easier to [copy it from my plugin](https://github.com/the-unrealist/specifier-reference-viewer/blob/main/Source/ReferenceGenerator/ReferenceGenerator.ubtplugin.csproj) and then edit the `EngineDir`, `GeneratorName`, and `RootNamespace` properties.

The Visual Studio solution file (`.sln`) is not necessary. The C# project will be rebuilt each time the game is built.

### 5. Create the exporter
Next, create a class and static method to serve as the main entry point for the exporter.

The class must have the `[UnrealHeaderTool]` attribute, and the exporter's method must have the `[UhtExporter]` attribute. The `Name` and `ModuleName` properties are required.

```csharp
using EpicGames.UHT.Tables;
using EpicGames.UHT.Utils;

[UnrealHeaderTool]
public static class Exporter
{
    [UhtExporter(Name = "MyCustomExporter", ModuleName = "MyCustomExporter", Options = UhtExporterOptions.Default)]
    public static void Generate(IUhtExportFactory factory)
    {
        // Process parsed code via factory.Session here.
        factory.Session.LogInfo("MyCustomExporter executed!");
    }
}
```

`UhtExporterOptions.Default` indicates that this exporter will be automatically executed each time UBT executes. Remove it or use `UhtExporterOptions.None` to make it opt-in just like the [sample exporters](#exporters).

### 6. Build
Build the game. Logs emitted by your exporter will appear in the build output.

```
0>  Running Internal UnrealHeaderTool C:\Path\To\MyGame.uproject C:\Path\To\MyGame\Intermediate\Build\Win64\MyGameEditor\Development\MyGameEditor.uhtmanifest -WarningsAsErrors -installed
0>Total of 0 written
0>C:\UE_5.3\Engine\Source\UnknownSource(1): Info: MyCustomExporter executed!
```
