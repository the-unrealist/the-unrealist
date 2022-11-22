---
layout: post
tags: unreal blueprints
title: "Custom Blueprint Nodes"
categories: [Blueprints]
author: Matt
excerpt: "A reference guide to creating custom Blueprint nodes in C++."
---

<img src="https://img.shields.io/badge/Unreal%20Engine-5.0-informational" alt="Written for Unreal Engine 5.0"> <img src="https://img.shields.io/badge/-Blueprints-blue" alt="Blueprints">

There's this fantastic [tutorial](https://www.gamedev.net/tutorials/programming/engines-and-middleware/improving-ue4-blueprint-usability-with-custom-nodes-r5694/) on creating custom Blueprint nodes. This article is intended as a supplement to the tutorial by providing additional information and reference tables, and for that reason, I recommend everyone to read the tutorial first.

# Table of Contents
1. [Introduction](#introduction)
2. [Graph Compatibility](#graph-compatibility)
   - [Check for Event Graph](#check-for-event-graph)
   - [Check for Functions](#check-for-functions)
   - [Require World Context](#require-world-context)
   - [Blueprint Derives From a Class](#blueprint-derives-from-a-class)
   - [Blueprint Implements Interface](#blueprint-implements-an-interface)
3. [Node Customization](#node-customization)
   - [Title](#title)
   - [Menu Category](#menu-category)
   - [Can Rename Node](#can-rename-node)
   - [Text Caching](#text-caching)

### More Sections Coming Soon
4. Pins
    - Creating Pins
    - Owner Pin
    - World Context Pin
    - User Defined Pins
    - Changing pins based on value
5. `ExpandNode`
6. Latent Nodes
7. Node Attributes
8. Tunnels and composite nodes
9. `UBlueprintNodeSpawner`

# Introduction
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

# Graph Compatibility
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

## Check for Construction Script
Construction scripts execute in the editor, so you may need this if your node is intended to not execute at edit time.
```cpp
bool bIsConstructionScript = FBlueprintEditorUtils::FindUserConstructionScript(Blueprint) == TargetGraph;
```

## Check for Event Graph
If your node expands into multiple distinct nodes (i.e. events), then you need to require it to be placed in only Event Graphs.
```cpp
bool bIsEventGraph = TargetGraph->GetSchema()->GetGraphType(TargetGraph) == GT_Ubergraph;
```

## Check for Functions
If your node expands into latent actions, then you need to prevent it from being placed in functions.
```cpp
bool bIsFunction = TargetGraph->GetSchema()->GetGraphType(TargetGraph) == GT_Function;
```

## Check for Macros
Unlike Event Graphs, macros can only have one input node. If your node expands into multiple input nodes, then you need to prevent it from being placed in macros.
```cpp
bool bIsMacro = TargetGraph->GetSchema()->GetGraphType(TargetGraph) == GT_Macro;
```

## Require World Context
Blueprint function libraries don't have a world context. If your node requires a world context, then you may want to check for this. Alternatively, you can expose the World Context pin as needed. Read more about this in the Pins section.
```cpp
bool bHasWorldContext = Blueprint->GeneratedClass->GetDefaultObject()->ImplementsGetWorld();
```

## Blueprint Derives From a Class
If your node is relevant only to a class, you can check to see if the Blueprint is derived from a class.
```cpp
bool bIsValidSubclass = Blueprint->ParentClass && Blueprint->ParentClass->IsChildOf(UMyClass::StaticClass());
```

## Blueprint Implements an Interface
Unlike `Blueprint->GeneratedClass->ImplementsInterface(...)`, the following code will detect interfaces added via the Blueprint editor — even before the Blueprint has been compiled.
```cpp
TArray<UClass*> ImplementedInterfaces;
FBlueprintEditorUtils::FindImplementedInterfaces(Blueprint, true, ImplementedInterfaces);
bool bImplementsInterface = ImplementedInterfaces.Contains(UBlendableInterface::StaticClass());
```

# Pins
Coming soon

# Node Customization
There are various functions that can be overriden to customize how your node appears in graphs and menus.

## Title
The title of the node. This can vary based on where it's being displayed. By default, this is the class name for all title types.
```cpp
virtual FText GetNodeTitle(ENodeTitleType::Type TitleType) const override;
```
<table>
   <thead>
      <tr><th>Title Type</th><th>Description</th></tr>
   </thead>
   <tbody>
      <tr><td><code>FullTitle</code></td><td>Displayed on the node in graphs. This can have multiple lines.</td></tr>
      <tr><td><code>ListView</code></td><td>A more concise, single-line title. Displayed in search results including "Find References".</td></tr>
      <tr><td><code>EditableTitle</code></td><td>For nodes that can be renamed, this should be the user-provided value.</td></tr>
      <tr><td><code>MenuTitle</code></td><td>Displayed in the Blueprint actions menu and context menus.</td></tr>
   </tbody>
</table>

## Menu Category
The category to put the node under in the Blueprint actions menu. By default or when it's an empty value, the action is placed under the top-level category. Subcategories are created by using the pipe `|` character as a delimiter, i.e. `"Category|Subcategory"`.
```cpp
virtual FText GetMenuCategory() const override;
```

## Can Rename Node
`GetCanRenameNode` can be overridden to enable users to rename nodes. You must also override `MakeNameValidator` function to provide a name validator or it will cause the editor to crash. You'll need to store the value in a field in `OnRenameNode` and return it in `GetNodeTitle` when the title type is `EditableTitle`.
```cpp
// K2Node_Custom.h

public:
    virtual bool GetCanRenameNode() const override;
    virtual FText GetNodeTitle(ENodeTitleType::Type TitleType) const override;
    virtual void OnRenameNode(const FString& NewName) override;
    virtual TSharedPtr<INameValidatorInterface> MakeNameValidator() const override;

private:
    UPROPERTY()
    FString UserDefinedTitle;
```
```cpp
// K2Node_Custom.cpp

bool UK2Node_Custom::GetCanRenameNode() const
{
#if WITH_EDITORONLY_DATA
    return true;
#else
    return false;
#endif
}

FText UK2Node_Custom::GetNodeTitle(ENodeTitleType::Type TitleType) const
{    
    if (TitleType == ENodeTitleType::EditableTitle && !UserDefinedTitle.IsEmpty())
    {
        return FText::FromString(UserDefinedTitle);
    }
    
    return NSLOCTEXT("Citrus", "NodeTitle", "Custom Node Title");
}

void UK2Node_Custom::OnRenameNode(const FString& NewName)
{
    UserDefinedTitle = NewName;
}

TSharedPtr<INameValidatorInterface> UK2Node_Custom::MakeNameValidator() const
{
    // Use the default name validator. Custom validators will need to derive from FKismetNameValidator.
    return MakeShareable(new FKismetNameValidator(GetBlueprint()));
}
```

## Text Caching
These functions are frequently called and generating a `FText` can be expensive. For this reason, it's recommended to cache text with `FNodeTextCache`. Some user actions like changing the input pin connections will automatically mark the cache as dirty. If needed, you can refresh the cache with the `MarkAsDirty` function.
```cpp
// K2Node_Custom.h

private:
    FNodeTextCache NodeTitleCache;
```
```cpp
// K2Node_Custom.cpp

FText UK2Node_Custom::GetNodeTitle(ENodeTitleType::Type TitleType) const
{
    if (NodeTitleCache.IsOutOfDate(this))
    {
        NodeTitleCache.SetCachedText(NSLOCTEXT("Citrus", "NodeTitle", "Custom Node Title"), this);
    }
    return NodeTitleCache;
}
```
