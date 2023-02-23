---
layout: post
tags: unreal component component-visualizers
title: "Visualize Actor Components in the Editor with Component Visualizers"
categories: [Unreal Editor]
author: Matt
excerpt: "Visualize actor components without physical representation in the Unreal Editor with custom Component Visualizers."
---

<img src="https://img.shields.io/badge/Unreal%20Engine-5.1-informational" alt="Written for Unreal Engine 5.1"> <img src="https://img.shields.io/badge/-C%2B%2B-orange" alt="C++">

## Table of Contents
- [Introduction](#introduction)
- [Getting Started](#getting-started)
- [Create the Component Visualizer](#create-the-component-visualizer)
- [Register the Component Visualizer](#register-the-component-visualizer)
- [Primitive Drawing Functions](#primitive-drawing-functions)
  - [Examples](#examples)
    - [`DrawPoint`](#drawpoint)
    - [`DrawLine`](#drawline)
    - [`DrawTranslucentLine`](#drawtranslucentline)
    - [`DrawFlatArrow`](#drawflatarrow)
    - [`DrawDirectionalArrow`](#drawdirectionalarrow)
  - [Reference List](#drawing-functions-reference-list)

## Introduction
Working with actor components that don't have a physical representation may be challenging. Recently, I learned about Component Visualizers which makes it possible to draw anything in the Unreal Editor for each component when selected.

In my sandbox construction game, each building piece has a set of polar connection points that may be either positive or negative. Two points with the same polarity cannot be attached to each other. That is, a positive point may only be attached to a negative point and vice-versa. Each connection point is represented by a custom Scene Component that has a polarity property. There are three relevant properties here: Location, Rotation, and Polarity. It would be helpful if I can visualize all three properties as a colored arrow, but only in the Unreal Editor when I'm designing a building piece.

Fortunately, Unreal Engine makes it easy to do this with [`FComponentVisualizer`](https://docs.unrealengine.com/5.1/en-US/API/Editor/UnrealEd/FComponentVisualizer/). Component visualizers are drawn when a component is selected.

## Getting Started
Component visualizers must be in an editor-only module that's loaded after the engine has been initialized. Create one if you do not have one yet.

```jsonc
// File: MyGame.uproject

"Modules": [
  {
    "Name": "MyGame",
    "Type": "Runtime",
    "LoadingPhase": "Default"
  },
  {
    "Name": "MyGameEditor",
    "Type": "Editor",
    "LoadingPhase": "PostEngineInit"
  }
]
```

Add `UnrealEd` and `ComponentVisualizers` to the editor module's dependencies.

```csharp
// File: MyGameEditor.Build.cs

PublicDependencyModuleNames.AddRange(
    new string[]
    {
        "Core",
        "MyGame"
    }
);

PrivateDependencyModuleNames.AddRange(
    new string[]
    {
        "CoreUObject",
        "Engine",
        "UnrealEd",
        "ComponentVisualizers"
    }
);
```

## Create the Component Visualizer

In the editor module, create a class derived from `FComponentVisualizer`. Override either `DrawVisualization` or `DrawVisualizationHUD` depending on whether the visualization renders inside the scene or on the editor's viewport.

```cpp
// File: MyComponentVisualizer.h

#pragma once

#include "ComponentVisualizer.h"

class FMyComponentVisualizer : public FComponentVisualizer
{
public:
    // Override this to draw in the scene
    virtual void DrawVisualization(const UActorComponent* Component, const FSceneView* View,
        FPrimitiveDrawInterface* PDI) override;
	
    // Override this to draw on the editor's viewport
    virtual void DrawVisualizationHUD(const UActorComponent* Component, const FViewport* Viewport,
        const FSceneView* View, FCanvas* Canvas) override;
};
```

```cpp
// File: MyComponentVisualizer.cpp

#include "MyComponentVisualizer.h"
#include "MyComponent.h"

void FMyComponentVisualizer::DrawVisualization(const UActorComponent* Component, const FSceneView* View,
    FPrimitiveDrawInterface* PDI)
{
    // Draw a visualization here using PDI (or Canvas if using DrawVisualizationHUD)
}
```

`FPrimitiveDrawInterface` provides basic drawing functions such as `DrawLine` and `DrawMesh`. Utility functions for drawing boxes, sphere, torus, and other advanced shapes are also available. Check out [Primitive Drawing Functions](#primitive-drawing-functions) for a comprehensive list and examples.

⚠️ The component's **absolute** location and rotation should be used in the drawing functions. If the relative location is used, then the visualization will be rendered incorrectly when the component's owning actor is selected in the level editor.

## Register the Component Visualizer
The component visualizer is registered in `StartupModule` and unregistered in `ShutdownModule`. Call `RegisterComponentVisualizer` with the name of the component that should be visualized.

```cpp
// File: MyGameEditor.cpp

#include "MyGameEditor.h"

#include "MyComponent.h"
#include "MyComponentVisualizer.h"
#include "UnrealEdGlobals.h"
#include "Editor/UnrealEdEngine.h"

#define LOCTEXT_NAMESPACE "FMyGameEditorModule"

void FMyGameEditorModule::StartupModule()
{
    if (GUnrealEd)
    {
        TSharedPtr<FMyComponentVisualizer> Visualizer = MakeShareable(new FMyComponentVisualizer());
        GUnrealEd->RegisterComponentVisualizer(UMyComponent::StaticClass()->GetFName(), Visualizer);
        Visualizer->OnRegister();
    }
}

void FMyGameEditorModule::ShutdownModule()
{
    if (GUnrealEd)
    {
        GUnrealEd->UnregisterComponentVisualizer(UMyComponent::StaticClass()->GetFName());
    }
}

#undef LOCTEXT_NAMESPACE
    
IMPLEMENT_MODULE(FMyGameEditorModule, MyGameEditor)
```

That's it! :)

## Primitive Drawing Functions
### Examples
#### `DrawPoint`
<img src="/assets/images/component-visualizer-draw-point.png" alt="A screenshot of a yellow point being drawn at the component's location.">

```cpp
void FMyComponentVisualizer::DrawVisualization(const UActorComponent* Component, const FSceneView* View,
    FPrimitiveDrawInterface* PDI)
{
    const UMyComponent* MyComponent = Cast<UMyComponent>(Component);
    if (!MyComponent)
    {
        return;
    }

    double Thickness = 15;
    PDI->DrawPoint(MyComponent->GetComponentLocation(), FLinearColor::Yellow, Thickness, SDPG_World);
}
```

#### `DrawLine`
<img src="/assets/images/component-visualizer-draw-line.jpg" alt="A screenshot of a yellow line being drawn on the X axis.">

```cpp
void FMyComponentVisualizer::DrawVisualization(const UActorComponent* Component, const FSceneView* View,
    FPrimitiveDrawInterface* PDI)
{
    const UMyComponent* MyComponent = Cast<UMyComponent>(Component);
    if (!MyComponent)
    {
        return;
    }

    double Length = 100;
    double Thickness = 1;
	
    FVector Start = MyComponent->GetComponentLocation();
    FVector End = Start + FRotationMatrix(MyComponent->GetComponentRotation()).GetScaledAxis(EAxis::X) * Length;

    PDI->DrawLine(Start, End, FLinearColor::Yellow, SDPG_World, Thickness);
}
```

#### `DrawTranslucentLine`

```cpp
void FMyComponentVisualizer::DrawVisualization(const UActorComponent* Component, const FSceneView* View,
    FPrimitiveDrawInterface* PDI)
{
    const UMyComponent* MyComponent = Cast<UMyComponent>(Component);
    if (!MyComponent)
    {
        return;
    }

    double Length = 100;
    double Thickness = 1;
	
    FVector Start = MyComponent->GetComponentLocation();
    FVector End = Start + FRotationMatrix(MyComponent->GetComponentRotation()).GetScaledAxis(EAxis::X) * Length;

    FLinearColor Color(1.0, 1.0, 0.0, 0.5); // RGBA in floating-point format (between 0 and 1)
    PDI->DrawTranslucentLine(Start, End, Color, SDPG_World, Thickness);
}
```

#### `DrawFlatArrow`
<img src="/assets/images/draw-flat-arrow.png" alt="A screenshot of a yellow flat arrow drawn along the X axis.">

```cpp
void FMyComponentVisualizer::DrawVisualization(const UActorComponent* Component, const FSceneView* View,
    FPrimitiveDrawInterface* PDI)
{
    const UMyComponent* MyComponent = Cast<UMyComponent>(Component);
    if (!MyComponent)
    {
        return;
    }

    FVector ComponentLocation = MyComponent->GetComponentLocation();
    FRotationMatrix ComponentRotation = FRotationMatrix(MyComponent->GetComponentRotation());
    FColor Color = FColor::Yellow;
    float Length = 100.f;
    float Width = 20.f;
    float Thickness = 1.f;

    DrawFlatArrow(PDI, ComponentLocation,
        ComponentRotation.GetScaledAxis(EAxis::X),
        ComponentRotation.GetScaledAxis(EAxis::Y),
        Color,
        Length,
        Width,
        GEngine->GeomMaterial->GetRenderProxy(),
        SDPG_World,
        Thickness);
}
```

#### `DrawDirectionalArrow`
<img src="/assets/images/directional-arrow.png" alt="A screenshot of a yellow 3D arrow drawn along the X axis.">

```cpp
void FMyComponentVisualizer::DrawVisualization(const UActorComponent* Component, const FSceneView* View,
    FPrimitiveDrawInterface* PDI)
{
    const UMyComponent* MyComponent = Cast<UMyComponent>(Component);
    if (!MyComponent)
    {
        return;
    }

    FLinearColor Color = FLinearColor::Yellow;
    float Length = 100.f;
    float Width = 20.f;
    float Thickness = 1.f;

    FMatrix Matrix = FScaleRotationTranslationMatrix(
        MyComponent->GetComponentScale(),
        MyComponent->GetComponentRotation(),
        MyComponent->GetComponentLocation());
	
    DrawDirectionalArrow(PDI, Matrix, Color, Length, Width, SDPG_World, Thickness);
}
```

### Drawing Functions Reference List
#### Primitive Drawing Interface
- [`DrawPoint`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/FPrimitiveDrawInterface/DrawPoint/)
- [`DrawLine`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/FPrimitiveDrawInterface/DrawLine/)
- [`DrawTranslucentLine`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/FPrimitiveDrawInterface/DrawTranslucentLine/)
- [`DrawSprite`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/FPrimitiveDrawInterface/DrawSprite/)
- [`DrawMesh`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/FPrimitiveDrawInterface/DrawMesh/)

#### Primitive Utility Functions
To set the color for most of the geometry functions in this list, you'll need to use `FDynamicColoredMaterialRenderProxy`. This will require adding `RenderCore` to your module's dependencies.

```cpp
auto* Proxy = new FDynamicColoredMaterialRenderProxy(GEngine->GeomMaterial->GetRenderProxy(), FLinearColor::Yellow);
PDI->RegisterDynamicResource(Proxy);

DrawPlane10x10(PDI, Plane, Radii, UVMin, UVMax, Proxy, SDPG_World);
```

- [`DrawPlane10x10`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawPlane10x10/)
- [`DrawTriangle`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawTriangle/)
- [`DrawBox`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawBox/)
- [`DrawSphere`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawSphere/)
- [`DrawCone`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawCone/)
- [`DrawCylinder`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawCylinder/)
- `DrawTorus`
- [`DrawDisc`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawDisc/)
- `DrawRectangleMesh`
- [`DrawFlatArrow`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawFlatArrow/)
- [`DrawWireBox`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawWireBox/)
- [`DrawCircle`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawCircle/)
- [`DrawArc`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawArc/)
- `DrawRectangle`
- [`DrawWireSphere`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawWireSphere/)
- [`DrawWireSphereAutoSides`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawWireSphereAutoSides/)
- [`DrawWireCylinder`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawWireCylinder/)
- [`DrawWireCapsule`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawWireCapsule/)
- [`DrawWireChoppedCone`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawWireChoppedCone/)
- [`DrawWireCone`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawWireCone/)
- [`DrawWireSphereCappedCone`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawWireSphereCappedCone/)
- [`DrawOrientedWireBox`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawOrientedWireBox/)
- [`DrawDirectionalArrow`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawDirectionalArrow/)
- [`DrawConnectedArrow`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawConnectedArrow/)
- [`DrawWireStar`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawWireStar/)
- [`DrawDashedLine`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawDashedLine/)
- [`DrawWireDiamond`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawWireDiamond/)
- [`DrawCoordinateSystem`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawCoordinateSystem/)
- [`DrawFrustumWireframe`](https://docs.unrealengine.com/5.1/en-US/API/Runtime/Engine/DrawFrustumWireframe/)
