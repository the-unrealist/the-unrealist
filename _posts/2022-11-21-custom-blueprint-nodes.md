---
layout: post
tags: unreal blueprints
title: "Custom Blueprint Nodes"
categories: [Blueprints]
author: Matt
excerpt: "A reference guide to creating custom Blueprint nodes in C++."
---

There's this fantastic [tutorial](https://www.gamedev.net/tutorials/programming/engines-and-middleware/improving-ue4-blueprint-usability-with-custom-nodes-r5694/) on creating custom Blueprint nodes. This article is intended as a supplement to the tutorial by providing additional information and reference tables, and for that reason, I recommend everyone to read the tutorial first.

## Table of Contents
1. [Introduction](#introduction)
2. [Graph Compatibility](#graph-compatibility)
   - [Check for Event Graph](#check-for-event-graph)
   - [Check for Functions](#check-for-functions)
   - [Require World Context](#require-world-context)
   - [Blueprint Derives From a Class](#blueprint-derives-from-a-class)
   - [Blueprint Implements Interface](#blueprint-implements-an-interface)

#### More Sections Coming Soon
4. Pins
    - Creating Pins
    - Owner Pin
    - World Context Pin
    - User Defined Pins
    - Changing pins based on value
5. Node Appearance
6. `ExpandNode`
7. Latent Nodes
8. Node Attributes
9. Tunnels and composite nodes
10. `UBlueprintNodeSpawner`

## Introduction
The Unreal Editor provides a general-purpose graph system that is used by Blueprints, materials, Niagara, and other graph-based features. In this tutorial, we'll focus on `UK2Node` from which all Blueprint nodes are derived.

All Blueprint nodes need to be in a `UncookedOnly` module. Create a new module and set the `Type` to `UncookedOnly` in the `uproject` or `uplugin` file.

```json
"Modules": [
  {
    "Name": "MyPluginUncooked",
    "Type": "UncookedOnly",
    "LoadingPhase": "Default"
   }
]
```

The modules `BlueprintGraph` and `UnrealEd` need to be referenced in `<ModuleName>.Build.cs`.

```csharp
PrivateDependencyModuleNames.AddRange(
    new string[]
    {
        "CoreUObject",
        "Engine",
        "BlueprintGraph",
        "UnrealEd"
    }
);
```

The simplest node that can be placed in any Blueprint graph requires creating a class derived from `UK2Node` and then overriding `GetMenuActions` to add itself to the Blueprint action database. Unreal Engine automatically detects and calls `GetMenuActions` on all classes derived from `UK2Node`.

```cpp
// K2Node_Custom.h

#pragma once

#include "CoreMinimal.h"
#include "K2Node.h"
#include "K2Node_Custom.generated.h"

UCLASS()
class MYPLUGINUNCOOKED_API UK2Node_Custom : public UK2Node
{
    GENERATED_BODY()
    
public:
    virtual void GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar) const override;
};
```

```cpp
// K2Node_Custom.cpp

#include "K2Node_Custom.h"
#include "BlueprintActionDatabaseRegistrar.h"
#include "BlueprintNodeSpawner.h"

void UK2Node_Custom::GetMenuActions(FBlueprintActionDatabaseRegistrar& ActionRegistrar) const
{
    UClass* ActionKey = GetClass();
    if (ActionRegistrar.IsOpenForRegistration(ActionKey))
    {
        UBlueprintNodeSpawner* NodeSpawner = UBlueprintNodeSpawner::Create(GetClass());
        check(NodeSpawner);

        ActionRegistrar.AddBlueprintAction(ActionKey, NodeSpawner);
    }
}
```

It'll appear at the bottom of the actions list labeled as the class name. Clicking on it will spawn a default node without any pins.
<img src="/assets/images/empty_node.png" alt="A screenshot of a custom node in Blueprints. It has no pins.">

## Graph Compatibility
By default, you can spawn your node in any Blueprint graph including construction scripts, functions, and macros. Override `IsCompatibleWithGraph` to restrict placement on certain graphs or even Blueprint classes.

```cpp
// K2Node_Custom.h

virtual bool IsCompatibleWithGraph(UEdGraph const* Graph) const override;
```
  
```cpp
// K2Node_Custom.cpp

bool UK2Node_Custom::IsCompatibleWithGraph(const UEdGraph* TargetGraph) const 
{
    UBlueprint* Blueprint = FBlueprintEditorUtils::FindBlueprintForGraph(TargetGraph);
    if (!Blueprint)
    {
        return false;
    }

    // Don't let this node spawn in construction scripts
    bool bIsCompatible = FBlueprintEditorUtils::FindUserConstructionScript(Blueprint) != TargetGraph;

    return Super::IsCompatibleWithGraph(TargetGraph) && bIsCompatible;
}
```

You may want to use one or more of the following booleans in your implementation of `IsCompatibleWithGraph`:

### Check for Construction Script
Construction scripts execute in the editor, so you may need this if your node is intended to not execute at edit time.
```cpp
bool bIsConstructionScript = FBlueprintEditorUtils::FindUserConstructionScript(Blueprint) == TargetGraph;
```

### Check for Event Graph
If your node expands into multiple distinct nodes (i.e. events), then you need to require it to be placed in only Event Graphs.
```cpp
bool bIsEventGraph = TargetGraph->GetSchema()->GetGraphType(TargetGraph) == GT_Ubergraph;
```

### Check for Functions
If your node expands into latent actions, then you need to prevent it from being placed in functions.
```cpp
bool bIsFunction = TargetGraph->GetSchema()->GetGraphType(TargetGraph) == GT_Function;
```

### Check for Macros
Unlike Event Graphs, macros can only have one input node. If your node expands into multiple input nodes, then you need to prevent it from being placed in macros.
```cpp
bool bIsMacro = TargetGraph->GetSchema()->GetGraphType(TargetGraph) == GT_Macro;
```

### Require World Context
Blueprint function libraries don't have a world context. If your node requires a world context, then you may want to check for this. Alternatively, you can expose the World Context pin as needed. Read more about this in the Pins section.
```cpp
bool bHasWorldContext = Blueprint->GeneratedClass->GetDefaultObject()->ImplementsGetWorld();
```

### Blueprint Derives From a Class
If your node is relevant only to a class, you can check to see if the Blueprint is derived from a class.
```cpp
bool bIsValidSubclass = Blueprint->ParentClass && Blueprint->ParentClass->IsChildOf(UMyClass::StaticClass());
```

### Blueprint Implements an Interface
```cpp
TArray<UClass*> ImplementedInterfaces;
FBlueprintEditorUtils::FindImplementedInterfaces(Blueprint, true, ImplementedInterfaces);
bool bImplementsInterface = ImplementedInterfaces.Contains(UBlendableInterface::StaticClass());
```
