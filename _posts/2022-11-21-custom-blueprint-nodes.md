---
layout: post
tags: unreal blueprints
title: "Custom Blueprint Nodes"
categories: [Blueprints]
author: Matt
excerpt: "A reference guide to creating custom Blueprint nodes in C++."
---

<img src="https://img.shields.io/badge/Unreal%20Engine-5.0-informational" alt="Written for Unreal Engine 5.0"> <img src="https://img.shields.io/badge/-Blueprints-blue" alt="Blueprints"> <img src="https://img.shields.io/badge/-C%2B%2B-orange" alt="C++">

## Introduction
The Unreal Editor provides a general-purpose graph system that is used by Blueprints, materials, Niagara, and other graph-based features. In this reference guide, we'll focus on `K2Node` from which all Blueprint nodes are derived.

There's [this fantastic tutorial on creating custom Blueprint nodes](https://www.gamedev.net/tutorials/programming/engines-and-middleware/improving-ue4-blueprint-usability-with-custom-nodes-r5694/). This page is intended as a supplement to the tutorial by providing additional information and reference tables, and for that reason, I recommend everyone to read the tutorial first.

## Table of Contents
1. [Create a Node](#create-a-node)
2. [Node Customization](#node-customization)
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
   - [Purity](#purity)
   - [Node Details](#node-details)
   - [Text Caching](#text-caching)
3. [Pins](#pins)
   - [Create Pins](#create-pins)
       - [Pin Categories](#pin-categories)
       - [Reserved Pin Names](#reserved-pin-names)
       - [Simple Example](#simple-example)
   - [Wildcard Pins](#wildcard-pins)
4. [Graph Compatibility](#graph-compatibility)
   - [Check for Construction Script](#check-for-construction-script)
   - [Check for Event Graph](#check-for-event-graph)
   - [Check for Function Graph](#check-for-function-graph)
   - [Check for Macro Graph](#check-for-macro-graph)
   - [Require World Context](#require-world-context)
   - [Blueprint Derives From a Class](#blueprint-derives-from-a-class)
   - [Blueprint Implements an Interface](#blueprint-implements-an-interface)

#### More Sections Coming Soon
4. Pins
    - Owner Pin
    - World Context Pin
    - User Defined Pins
    - Changing pins based on value
5. `ExpandNode`
6. Latent Nodes
7. Node Attributes
8. Tunnels and composite nodes
9. `UBlueprintNodeSpawner`

## Create a Node
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

### Purity
To make the compiler recognize this node as being pure, override `IsNodePure` to return `true`.
```cpp
virtual bool IsNodePure() const override { return true; }
```

### Node Details
If enabled, all visible properties appear in the details window when the node is selected. A property is visible when it has any of the "Edit" or "Visible" specifiers such as `EditAnywhere` or `VisibleAnywhere`.
```cpp
virtual bool ShouldShowNodeProperties() const override { return true; }
	
UPROPERTY(EditAnywhere, Category="My Custom Node")
UObject* CustomProperty;
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

## Pins
### Create Pins
`AllocateDefaultPins` is called whenever the node needs to be created for a multitude of reasons, i.e., spawning the node or reopening its containing Blueprint graph. Override this function and call `CreatePin` to create the input and output pins.

There are many overrides for `CreatePin` with different sets of parameters, but you'll likely use one of the following:
```cpp
UEdGraphPin* CreatePin(EEdGraphPinDirection Dir, const FName PinCategory, const FName PinName, const FCreatePinParams& PinParams = FCreatePinParams());
UEdGraphPin* CreatePin(EEdGraphPinDirection Dir, const FName PinCategory, const FName PinSubCategory, const FName PinName, const FCreatePinParams& PinParams = FCreatePinParams());
UEdGraphPin* CreatePin(EEdGraphPinDirection Dir, const FName PinCategory, UObject* PinSubCategoryObject, const FName PinName, const FCreatePinParams& PinParams = FCreatePinParams());
```

<table>
  <thead>
    <tr><th>Parameter</th><th>Description</th></tr>
  </thead>
  <tbody>
    <tr>
      <td><code>Dir</code></td>
      <td>Specifies whether this pin is an input (<code>EGPD_Input</code>) or output (<code>EGPD_Output</code>) pin.</td>
    </tr>
    <tr>
      <td><code>PinCategory</code></td>
      <td>This is the main type of the pin. Refer to the Pin Categories table below for a list of available categories.</td>
    </tr>
    <tr>
      <td><code>PinSubCategory</code></td>
      <td>The sub-type of the pin. This is used by a small number of pin types. Refer to the Pin Categories table below for more information.</td>
    </tr>
    <tr>
      <td><code>PinSubCategoryObject</code></td>
      <td>The sub-type of the pin using the provided object. For example, if the pin is a class reference, then this would be the class type.</td>
    </tr>
    <tr>
      <td><code>PinName</code></td>
      <td>A name for the pin. The name needs to be unique within the node. Certain names have special meaning. Refer to the Special Pin Names table below for more information.</td>
    </tr>
    <tr>
      <td><code>PinParams</code></td>
      <td>
          Additional parameters for the pin.<br/><br/>
          <table>
            <tbody>
              <tr><td><code>ContainerType</code></td><td>Sets whether this pin is an array, set, map, or a single value.</td></tr>
              <tr><td><code>bIsReference</code></td><td>Sets whether to pass the value as a reference.</td></tr>
              <tr><td><code>bIsConst</code></td><td>Sets whether the value is immutable.</td></tr>
              <tr><td><code>Index</code></td><td>Sets the position of this pin in the pin list. By default, pins are appended to the list.</td></tr>
              <tr><td><code>ValueTerminalType</code></td><td>Sets the value type if the container type is a map.</td></tr>
            </tbody>
          </table>
      </td>
    </tr>
  </tbody>
</table>

#### Pin Categories
Pin categories and subcategories are defined in `UEdGraphSchema_K2`.
<table>
  <thead>
    <tr><th>Name</th><th><code>PinCategory</code></th><th><code>PinSubCategory</code></th><th><code>PinSubCategoryObject</code></th></tr>
  </thead>
  <tbody>
    <tr><td>Exec</td><td><code>PC_Exec</code></td><td>-</td><td>-</td></tr>
    <tr><td>Boolean</td><td><code>PC_Boolean</code></td><td>-</td><td>-</td></tr>
    <tr><td>Byte</td><td><code>PC_Byte</code></td><td>For bitmask, use <code>PSC_Bitmask</code></td><td>-</td></tr>
    <tr><td>Integer</td><td><code>PC_Int</code></td><td>For bitmask, use <code>PSC_Bitmask</code></td><td>-</td></tr>
    <tr><td>Integer64</td><td><code>PC_Int64</code></td><td>-</td><td>-</td></tr>
    <tr><td>Float</td><td><code>PC_Real</code></td><td><table style="margin: 0;"><tbody><tr><td>Single-precision</td><td><code>PC_Float</code></td></tr><tr><td>Double-precision</td><td><code>PC_Double</code></td></tr></tbody></table></td><td>-</td></tr>
    <tr><td>Name</td><td><code>PC_Name</code></td><td>-</td><td>-</td></tr>
    <tr><td>String</td><td><code>PC_String</code></td><td>-</td><td>-</td></tr>
    <tr><td>Text</td><td><code>PC_Text</code></td><td>-</td><td>-</td></tr>
    <tr><td>Class Reference</td><td><code>PC_Class</code></td><td>For "self", use <code>PSC_Self</code></td><td><code>UClass*</code> for the class type</td></tr>
    <tr><td>Soft Class Reference</td><td><code>PC_SoftClass</code></td><td>-</td><td><code>UClass*</code> for the class type</td></tr>
    <tr><td>Object Reference</td><td><code>PC_Object</code></td><td>For "self", use <code>PSC_Self</code></td><td><code>UClass*</code> for the class type</td></tr>
    <tr><td>Soft Object Reference</td><td><code>PC_SoftObject</code></td><td>-</td><td><code>UClass*</code> for the class type</td></tr>
    <tr><td>Struct</td><td><code>PC_Struct</code></td><td>-</td><td><code>UScriptStruct*</code> for the struct type</td></tr>
    <tr><td>Enum</td><td><code>PC_Enum</code></td><td>-</td><td><code>UEnum*</code> for the enum type</td></tr>
    <tr><td>Delegate (Event)</td><td><code>PC_Delegate</code></td><td>-</td><td><code>UFunction*</code> for the function signature, or <code>nullptr</code> to accept any function/event</td></tr>
    <tr><td>Interface*</td><td><code>PC_Interface</code></td><td>-</td><td><code>UClass*</code> for the interface type</td></tr>
    <tr><td>Wildcard</td><td><code>PC_Wildcard</code></td><td>If the pin represents an index in a list, use <code>PSC_Index</code> to allow Integer, Bool, Byte, and Enum values</td><td>-</td></tr>
  </tbody>
</table>

*This pin accepts a reference to an object that implements the specified interface.

#### Reserved Pin Names
There are many "reserved" pin names defined in `UEdGraphSchema_K2`. Some of the common ones are:

<table>
  <thead>
    <tr><th>Name</th><th>Input / Output</th><th>Pin Category</th></tr>
  </thead>
  <tbody>
    <tr><td><code>PN_Execute</code><br/>Unnamed input exec pin</td><td>Input</td><td><code>PC_Exec</code></td></tr>
    <tr><td><code>PN_Then</code><br/>Unnamed output exec pin</td><td>Output</td><td><code>PC_Exec</code></td></tr>
    <tr><td><code>PN_ReturnValue</code><br/>The return object</td><td>Output</td><td><code>PC_Object</code></td></tr>
  </tbody>
</table>

#### Simple Example
Here's an example of a simple node. Examples of more advanced pins are provided in later sections.

<img src="/assets/images/example_node.png" alt="An example node with input and output exec pins and a floating point input pin">

```cpp
// K2Node_Custom.h

public:
    virtual void AllocateDefaultPins() override;
```
```cpp
// K2Node_Custom.cpp

void UK2Node_CustomBlueprintNode::AllocateDefaultPins()
{
    Super::AllocateDefaultPins();

    // Input exec pin
    CreatePin(EGPD_Input, UEdGraphSchema_K2::PC_Exec, UEdGraphSchema_K2::PN_Execute);

    // Output exec pin
    CreatePin(EGPD_Output, UEdGraphSchema_K2::PC_Exec, UEdGraphSchema_K2::PN_Then);

    // Example float input pin
    static FName ExamplePinName = TEXT("Some Value");
    CreatePin(EGPD_Input, UEdGraphSchema_K2::PC_Real, UEdGraphSchema_K2::PC_Double, ExamplePinName);
}
```

### Wildcard Pins
There's some work required to make wildcard pins behave as expected. If you create the pin and do nothing else, then the type of the pin will not change when another node is connected to it. This means we need to override `NotifyPinConnectionListChanged` to check the wildcard pin and set its type if it's connected to another node. This function is called each time a pin is connected or disconnected, so it's the perfect place to put our type checking logic.

In this example, we'll create a wildcard input and output pin. Our goal is to make the output pin type match the input pin type. By the way, it's a good practice to define the pin name outside the function but to make it simple, the pin names are inlined.
```cpp
// K2Node_Custom.cpp

void UK2Node_CustomBlueprintNode::AllocateDefaultPins()
{
    Super::AllocateDefaultPins();

    CreatePin(EGPD_Input, UEdGraphSchema_K2::PC_Exec, UEdGraphSchema_K2::PN_Execute);
    CreatePin(EGPD_Output, UEdGraphSchema_K2::PC_Exec, UEdGraphSchema_K2::PN_Then);

    CreatePin(EGPD_Input, UEdGraphSchema_K2::PC_Wildcard, TEXT("WildcardInput"));
    CreatePin(EGPD_Output, UEdGraphSchema_K2::PC_Wildcard, TEXT("WildcardOutput"));
}
```

Override `NotifyPinConnectionListChanged`. Let's begin by checking both the input and output pin and reset their type to wildcard if they're both disconnected. We want to check for both at the same time because we don't want to reset the pin type if either pin is already connected to another node. Next, if either wildcard pin is connected, we want to check the pin that was connected to it. If it's also a wildcard, then we still don't know the type and must do nothing. Finally, when the wildcard pin's type has changed, we change all other wildcard pins to have the same type and break any pin connections that are no longer valid.
```cpp
// K2Node_Custom.h

public:
    virtual void NotifyPinConnectionListChanged(UEdGraphPin* Pin) override;
```
```cpp
// K2Node_Custom.cpp

void UK2Node_CustomBlueprintNode::NotifyPinConnectionListChanged(UEdGraphPin* Pin)
{
    Super::NotifyPinConnectionListChanged(Pin);

    UEdGraphPin* InputPin = FindPin(TEXT("WildcardInput"));
    UEdGraphPin* OutputPin = FindPin(TEXT("WildcardOutput"));
    
    if (InputPin->LinkedTo.Num() == 0 && OutputPin->LinkedTo.Num() == 0)
    {
        // Reset input pin to wildcard
        InputPin->PinType.PinCategory = UEdGraphSchema_K2::PC_Wildcard;
        InputPin->PinType.PinSubCategory = TEXT("");
        InputPin->PinType.PinSubCategoryObject = nullptr;

        // Reset output pin to wildcard
        OutputPin->PinType.PinCategory = UEdGraphSchema_K2::PC_Wildcard;
        OutputPin->PinType.PinSubCategory = TEXT("");
        OutputPin->PinType.PinSubCategoryObject = nullptr;
    }
    else if ((Pin == InputPin || Pin == OutputPin) && Pin->LinkedTo.Num() > 0 && Pin->LinkedTo[0]->PinType.PinCategory != UEdGraphSchema_K2::PC_Wildcard)
    {
        // Set the wildcard pin type to the connected pin type.
        Pin->PinType = Pin->LinkedTo[0]->PinType;

        // Update all wildcard pins to have the same type.
        InputPin->PinType = Pin->PinType;
        OutputPin->PinType = Pin->PinType;

        // Break any connection if it's no longer valid.
        UEdGraphSchema_K2::ValidateExistingConnections(InputPin);
        UEdGraphSchema_K2::ValidateExistingConnections(OutputPin);
    }
}
```

This screenshot demonstrates the logic we implemented in `NotifyPinConnectionListChanged`:

<img src="/assets/images/wildcard_pins.png" alt="A screenshot of a custom node with wildcard pins">

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
