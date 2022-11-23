---
layout: post
tags: unreal blueprints
title: "Custom Blueprint Nodes"
categories: [Blueprints]
author: Matt
excerpt: "A reference guide to creating custom Blueprint nodes in C++."
---

<img src="https://img.shields.io/badge/Unreal%20Engine-5.0-informational" alt="Written for Unreal Engine 5.0"> <img src="https://img.shields.io/badge/-Blueprints-blue" alt="Blueprints"> <img src="https://img.shields.io/badge/-C%2B%2B-orange" alt="C++">

There's [this fantastic tutorial on creating custom Blueprint nodes](https://www.gamedev.net/tutorials/programming/engines-and-middleware/improving-ue4-blueprint-usability-with-custom-nodes-r5694/). This page is intended as a supplement to the tutorial by providing additional information and reference tables, and for that reason, I recommend everyone to read the tutorial first.

## Table of Contents
1. [Introduction](#introduction)
2. [Graph Compatibility](#graph-compatibility)
   - [Check for Construction Script](#check-for-construction-script)
   - [Check for Event Graph](#check-for-event-graph)
   - [Check for Function Graph](#check-for-function-graph)
   - [Check for Macro Graph](#check-for-macro-graph)
   - [Require World Context](#require-world-context)
   - [Blueprint Derives From a Class](#blueprint-derives-from-a-class)
   - [Blueprint Implements an Interface](#blueprint-implements-an-interface)
3. [Node Customization](#node-customization)
   - [Title](#title)
   - [Menu Category](#menu-category)
   - [Tooltips](#tooltips)
   - [Keywords](#keywords)
   - [Appearance](#appearance)
       - [Colors](#colors)
       - [Compact Node](#compact-node)
       - [Bead Node](#bead-node)
       - [Variable Node](#variable-node)
       - [Control Point / Knot](#control-point--knot)
       - [Icons](#icons)
   - [Can Rename Node](#can-rename-node)
   - [Text Caching](#text-caching)
   - [Purity](#purity)

#### More Sections Coming Soon
4. Pins
    - Creating Pins
    - Wildcard Pins
    - Owner Pin
    - World Context Pin
    - User Defined Pins
    - Changing pins based on value
5. `ExpandNode`
6. Latent Nodes
7. Node Attributes
8. Tunnels and composite nodes
9. `UBlueprintNodeSpawner`

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

### Check for Function Graph
If your node expands into latent actions, then you need to prevent it from being placed in functions.
```cpp
bool bIsFunction = TargetGraph->GetSchema()->GetGraphType(TargetGraph) == GT_Function;
```

### Check for Macro Graph
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
Unlike `Blueprint->GeneratedClass->ImplementsInterface(...)`, the following code will detect interfaces added via the Blueprint editor — even before the Blueprint has been compiled.
```cpp
TArray<UClass*> ImplementedInterfaces;
FBlueprintEditorUtils::FindImplementedInterfaces(Blueprint, true, ImplementedInterfaces);
bool bImplementsInterface = ImplementedInterfaces.Contains(UBlendableInterface::StaticClass());
```

## Node Customization
There are various functions that can be overriden to customize how your node appears in graphs and menus.

### Title
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

### Menu Category
The category to put the node under in the Blueprint actions menu. By default or when it's an empty value, the action is placed under the top-level category. Subcategories are created by using the pipe `|` character as a delimiter, i.e. `"Category|Subcategory"`.
```cpp
virtual FText GetMenuCategory() const override;
```

### Tooltips
The tooltip appears in both the graph and the Blueprint actions menu. The tooltip heading is the smaller text above the tooltip text. It's visible only after a node was placed. Unreal Editor uses it to display information of some status that may affect how the node behaves, such as "Replicated" when a variable is replicated.

<img src="/assets/images/node_tooltip.png" alt="A screenshot of a custom node with its tooltip visible.">

```cpp
virtual FText GetTooltipText() const override;
virtual FText GetToolTipHeading() const override;
```

### Keywords
This defines the keywords to help users find this action using the search box in the Blueprint actions menu.
```cpp
virtual FText GetKeywords() const override;
```

### Appearance

#### Colors
Override these functions to set the color of the node. `GetNodeTitleColor` provides the color for the title bar. `GetNodeBodyTintColor` is not used by any Blueprint node, but it does work if you want to make your node stand out!

<img src="/assets/images/colorful_nodes.png" alt="A series of nodes in various colors">

```cpp
virtual FLinearColor GetNodeTitleColor() const override;
virtual FLinearColor GetNodeBodyTintColor() const override;
```

#### Compact Node
This makes the title centered in the node and displayed in a larger font. To make the node compact, override `ShouldDrawCompact` to return `true`. By default, `GetCompactNodeTitle` uses the full title of the node, but you can override `GetCompactNodeTitle` to change this.

<img src="/assets/images/compact_nodes.png" alt="Examples of compact nodes (Add integers, subsystem, and boolean OR nodes)">

```cpp
virtual bool ShouldDrawCompact() const override;
virtual FText GetCompactNodeTitle() const override;
```

#### Bead Node
This node has no fixed location. It is always located in the middle between the input node and the output node. No Blueprint nodes use this as of Unreal Engine 5, and it appears to be a legacy option.
```cpp
virtual bool ShouldDrawAsBead() const override { return true; }
```

#### Variable Node
This makes the node appear as a variable node. In other words, the title is hidden and only the pins are visible.
```cpp
virtual bool DrawNodeAsVariable() const override { return true; }
```

#### Control Point / Knot
This makes the node appear as a knot like reroute nodes. Your node needs to have just one input and one output pin. You also need to provide the input and output pin (typically `0` and `1` respectively).
```cpp
virtual bool ShouldDrawNodeAsControlPointOnly(int32& OutInputPinIndex, int32& OutOutputPinIndex) const override
{
    OutInputPinIndex = 0;
    OutOutputPinIndex = 1;
    return true;
}
```

#### Icons
Override `GetIconAndTint` to set the icon that appears on the title bar. `ShowPaletteIconOnNode` controls whether this icon is visible. `GetCornerIcon` sets the icon that appears on the top-right corner of the node.
```cpp
virtual FSlateIcon GetIconAndTint(FLinearColor& OutColor) const override
{
     static const FSlateIcon Icon = FSlateIcon("EditorStyle", "GraphEditor.Default_16x");
     return Icon;
}
virtual bool ShowPaletteIconOnNode() const override { return false; }
virtual FName GetCornerIcon() const override { return TEXT("Graph.Latent.LatentIcon"); }
```

### Can Rename Node
Set `bCanRenameNode` to `1` to allow users to rename the node. Alternatively, you may override `GetCanRenameNode` to return `true`. Beware that `bCanRenameNode` exists only when the `WITH_EDITORONLY_DATA` flag exists, so you'll need to put it in between `#if WITH_EDITORONLY_DATA` and `#endif` just like in the example below.

You must also override `MakeNameValidator` function to provide a name validator or it will cause the editor to crash. You'll need to store the value in a field in `OnRenameNode` and return it in `GetNodeTitle` when the title type is `EditableTitle`.
```cpp
// K2Node_Custom.h

public:
    UK2Node_Custom();
    virtual FText GetNodeTitle(ENodeTitleType::Type TitleType) const override;
    virtual void OnRenameNode(const FString& NewName) override;
    virtual TSharedPtr<INameValidatorInterface> MakeNameValidator() const override;

private:
    UPROPERTY()
    FString UserDefinedTitle;
```
```cpp
// K2Node_Custom.cpp

UK2Node_Custom::UK2Node_Custom()
{
#if WITH_EDITORONLY_DATA
    bCanRenameNode = 1;
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

### Text Caching
Many of the customization functions mentioned in this reference guide are frequently called and generating a `FText` can be expensive. For this reason, it's recommended to cache text with `FNodeTextCache`. Some user actions like changing the input pin connections will automatically mark the cache as dirty. If needed, you can refresh the cache with the `MarkAsDirty` function.
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

### Purity
To make the compiler recognize this node as being pure, override `IsNodePure` to return `true`.
```cpp
virtual bool IsNodePure() const override;
```