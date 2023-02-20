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
- [Introduction](#Introduction)
- [Getting Started](#Getting-Started)
- [Create the Component Visualizer](#Create-the-Component-Visualizer)
- [Register the Component Visualizer](#Register-the-Component-Visualizer)
- [Primitive Drawing Functions](#Primitive-Drawing-Functions)

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

`FPrimitiveDrawInterface` provides basic drawing functions such as `DrawLine` and `DrawMesh`. Utility functions for drawing boxes, sphere, torus, and other advanced shapes are also available. Check out [Primitive Drawing Functions](#Primitive-Drawing-Functions) for a comprehensive list and examples.

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
This is a work-in-progress. Coming soon!
