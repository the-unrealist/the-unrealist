# Unreal Header Tool (UHT) Script Generator Plugins

I'll begin with an introduction to the Unreal Header Tool before I explain what script generator plugins are and how to create one.

## About the Unreal Header Tool
The Unreal Header Tool (UHT) generates code for `UObject` types. Many of the C++ macros in the Unreal Engine source code are actually implemented by the UHT.

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

The UHT generates [two files named `Example.generated.h` and `Example.gen.cpp`](https://gist.github.com/the-unrealist/0aa6b16d1a89c13cd0065b685b9a0bce) with code generated for the `UCLASS()` and `GENERATED_BODY()` macros. This is how Unreal implements [reflection](https://docs.unrealengine.com/5.3/en-US/unreal-object-handling-in-unreal-engine/#run-timetypeinformationandcasting) and [garbage collection](https://docs.unrealengine.com/5.3/en-US/unreal-object-handling-in-unreal-engine/#garbagecollection) in a language that does not support these. Furthermore, serialization, network replication, and editor integration are also implemented as generated code.

### Exporters
This C++ code generation is implemented as an **exporter** in the UHT.

An exporter processes the parsed code and then optionally *exports* files to the `Intermediate` directory. An exporter may serve as an analyzer and not export any files—as demonstrated in the `Stats` exporter sample.

UHT implements these exporters under `/Engine/Source/Programs/Shared/EpicGames.UHT/Exporters/`.

|Name|Enabled by default?|Purpose|
|----|-------------------|-------|
|CodeGen|Yes|Generates C++ code for the `UObject` system.|
|Json|No|A sample exporter that shows how to output files. This exporter generates a JSON description of each package.|
|Stats|No|A sample exporter that shows how to log details about types in the codebase.|

Exporters that are not enabled by default can be triggered by adding `-<EXPORTER NAME>` to the commandline arguments. Replace `MyGameEditor`, `Win64`, and `Development` with the desired target, platform, and build configuration. The following command will execute both the `Stats` and `Json` exporters.

```shell
C:\Program Files\Epic Games\UE_5.3\Engine\Build\BatchFiles>Build.bat MyGameEditor Win64 Development -Project="C:/Path/To/MyGame.uproject" -Stats -Json
```
