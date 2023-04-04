---
tags: 
  - unreal
  - Lyra
  - experiences
title: "Lyra Deep Dive - Chapter 2: Experiences"
categories: 
  - Lyra
excerpt: "Lyra introduces the concept of Experiences which are essentially modular game modes. In this chapter, we'll walk through how the Experiences system is implemented."
---

<img src="https://img.shields.io/badge/Unreal%20Engine-5.1-informational" alt="Written for Unreal Engine 5.1"> <img src="https://img.shields.io/badge/-C%2B%2B-orange" alt="C++">

This is the second chapter in the [Lyra Deep Dive](https://unrealist.org/lyra-part-1) series.

## What are Experiences?
TODO: Explain what Lyra Experiences are and be sure to mention that most of the code for this feature is in the `GameModes` folder.

## Source Code
[View the source code for this chapter ❭](https://github.com/the-unrealist/lyra-deep-dive/tree/chapter2-experiences)

I recommend looking at the [diff](https://github.com/the-unrealist/lyra-deep-dive/compare/chapter1-introduction...chapter2-experiences) to see what's changed since the previous chapter.

## Plugins
The Lyra Experiences system is driven by the combination of the **Game Features** and **Modular Gameplay** plugins. With the Game Features plugin, experiences are contained as standalone game feature plugins and loaded on demand. The Modular Gameplay plugin allows experiences to add components to actors, modify game state, add data sources, and much more.

### Game Features
TODO: Provide an overview of the Game Features plugin.

### Modular Gameplay
The Modular Gameplay plugin enables actors to register themselves as receivers for components. It's usually used in conjunction with the Game Features plugin to make actors extensible via plugins and avoid coupling the actor with the feature.

Actors you want to make modular will need to register themselves with the `UGameFrameworkComponentManager`. This is typically done in the `PreInitializeComponents` function of the actor.

```cpp
void AMyModularActor::PreInitializeComponents()
{
    Super::PreInitializeComponents();
    UGameFrameworkComponentManager::AddGameFrameworkComponentReceiver(this);
}
```

Actors also need to unregister themselves when they're no longer accepting extensions. This is typically done in the `EndPlay` function of the actor.

```cpp
void AMyModularActor::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
    UGameFrameworkComponentManager::RemoveGameFrameworkComponentReceiver(this);
    Super::EndPlay(EndPlayReason);
}
```

Modular actors can create custom extension points by broadcasting custom events to any object listening to the actor.

Most modular actors will want to send the `GameActorReady` event in `BeginPlay` so that listeners can execute code only when the actor is active.

```cpp
void AMyModularActor::BeginPlay()
{
    UGameFrameworkComponentManager::SendGameFrameworkComponentExtensionEvent(this, UGameFrameworkComponentManager::NAME_GameActorReady);
    Super::BeginPlay();
}
```

To handle actor extensions, call `AddExtensionHandler` in the `UGameFrameworkComponentManager` subsystem. This will subscribe to all actors of the desired class for extension events. It must be an actor subclass and not the root `AActor` class, which means you _cannot_ subscribe to every single actor. This is intentionally checked to prevent significant performance impact to the game.

```cpp
// This snippet is placed where you want to start handling actor extensions.
if (UGameFrameworkComponentManager* ComponentManager = UGameInstance::GetSubsystem<UGameFrameworkComponentManager>(GameInstance))
{			
    TSharedPtr<FComponentRequestHandle> ExtensionRequestHandle = ComponentManager->AddExtensionHandler(
        AMyModularActor::StaticClass(),
        UGameFrameworkComponentManager::FExtensionHandlerDelegate::CreateUObject(this, &ThisClass::HandleActorExtension));
}
```

Store `ExtensionRequestHandle` somewhere. Call `Unregister` on it to stop listening for extension events.

In the handler delegate, you typically would check the event name and execute code on the actor if it's an event you wish to handle.

```cpp
void UMyActorExtensionHandler::HandleActorExtension(AActor* Actor, FName EventName)
{
    if (EventName == UGameFrameworkComponentManager::NAME_GameActorReady)
    {
        // Do something when the actor is ready.
    }
    
    // Handle other extension events here if needed.
}
```

### Modular Gameplay Actors
The `ModularGameplayActors` plugin contains subclasses of the [standard gameplay framework actors](https://docs.unrealengine.com/5.1/en-US/gameplay-framework-quick-reference-in-unreal-engine/) that are registered for extension via the Modular Gameplay plugin. While you can always register directly with `UGameFrameworkComponentManager` in your actors, this plugin makes it so that you don't need to do it manually.

These actors are provided by this plugin:
* `AModularAIController`
* `AModularCharacter`
* `AModularGameModeBase`
* `AModularGameMode`
* `AModularGameStateBase`
* `AModularGameState`
* `AModularPawn`
* `AModularPlayerController`
* `AModularPlayerState`

This plugin does not come with Unreal Engine out of the box, so we need to manually create it. You can find this plugin in [the source code for this chapter](https://github.com/the-unrealist/lyra-deep-dive/tree/chapter2-experiences/LyraStarterGame/Plugins/ModularGameplayActors).

## Experience Definitions
### Experience Definition Data Asset
TODO

### Action Sets
TODO

### User-Facing Experience Definition Data Asset
TODO

## Replicated Experience
TODO: explain how an experience is replicated to all players. By the end of this section, the experience definition is replicated to all clients when changed.

### Experience Manager Component
TODO

### Lyra Game State
TODO

### Lyra Asset Manager
TODO

## Experience Asset Loading
TODO: Explain how an experience is loaded. By the end of this section, the experience definition asset and asset sets are loaded, and then all game feature plugins referenced by the definition and any asset sets are loaded too.

### Experience Manager
TODO

### Experience Manager Component
TODO

## Execution
TODO: Execute the experience definition's game feature actions and notify events.

### Experience Manager Component
TODO

## Deactivation
TODO

## Asset Manager
TODO: Primary asset types to scan in project settings

## Pawn Spawning and World Settings
### Lyra Game Mode
TODO

### Lyra World Settings
TODO

## Default Experience
TODO: Create default Lyra experience definition, and set world settings.

## Blueprint Nodes
TODO: Talk about `AsyncAction_ExperienceReady` and any other blueprint nodes related to Lyra Experiences.
