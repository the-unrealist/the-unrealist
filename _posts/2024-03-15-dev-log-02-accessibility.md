---
tags: 
  - unreal
title: "Dev Log 02: How I'm Making My Game Accessible"
categories:
  - Dev Logs
---
<img src="https://img.shields.io/badge/Unreal%20Engine-5-informational" alt="Written for Unreal Engine 5"> <img src="https://img.shields.io/badge/-Dev%20Log-red" alt="Dev Log"> <img src="https://img.shields.io/badge/-C%2B%2B-orange" alt="C++">

Did you know that **[one in five casual gamers have a disability](https://www.gamesindustry.biz/popcap-games-research-publisher-s-latest-survey-says-that-casual-games-are-big-with-disabled-people)**—myself included? I'm creating a cozy life simulation game, so players with disabilities make up a significant portion of my target audience. It's clear that my game must be accessible.

There are [many ways to make a game accessible](https://gameaccessibilityguidelines.com/), and I strive to implement as much as I reasonably can.

— But that's not the focus of this article.

Retrofitting a game with accessible features is hard, and this is why many games end up with just a band-aid solution. **I want to share *how* I'm setting up my game to be accessible in a modular way—as Game Feature plugins.**

## Accessibility as a Game Feature plugin?
Yes. And it works really well!

<details open style="margin-bottom: 1em;">
  <summary class="toggle-link" style="cursor: pointer;">Toggle animated preview</summary>
  <img src="/assets/images/accessibility-plugin2.gif" style="height: 400px;" alt="The high contrast and large text settings are applied to a dialog box when the Game Feature is activated. It reverts to the default styling when deactivated.">
</details>

Just to be clear, I don't mean *game features* as a concept. I am referring to the [Game Features system](https://docs.unrealengine.com/5.3/en-US/game-features-and-modular-gameplay-in-unreal-engine/), which is a relatively new Unreal Engine feature that makes it possible to create standalone content for a game.

In my game, each accessibility setting is implemented as a Game Feature plugin for these reasons:
* It enables previewing UMG widgets with accessible settings at design time.
* Some settings require assets that I don't want loaded when they're not in use, such as [alternative fonts](https://opendyslexic.org/), additional icons, and replacement textures.
* Compartmentalization makes it easy to add (and maintain) accessibility settings.
* The Game Features system has a built-in activation/deactivation mechanism that also works in the editor.
* The only responsibility of a save game system is to activate the correct plugin based on user preferences.

To make all of this work, the game not only needs to know about these settings, but also needs to do so in a modular manner.

To solve this problem, I created the Global Parameters plugin that integrates with the Game Features system. A custom Game Feature action overrides global parameters when activated.

## Global Parameters
A variable that's used as input for some function or calculation is a parameter.

Global variables are generally a sign of poor design, but global *parameters* are appropriate for some purposes such as:
* feature flags
* accessibility overrides
* UI theme system (dark mode, maybe? 🌚)

### Querying
Gameplay tags are used for parameter names.

Any `UObject` may query a global parameter. All accessors have a default fallback value for when the parameter is not set.

<img src="/assets/images/dev-log-02-accessors.png" alt="Pure Blueprint nodes for getting the value from global flag, color, brush, font, and scalar parameters.">

### Parameter collections
Parameters are defined in a custom data asset, `USunParameterCollection`. This is similar to `UMaterialParameterCollection`, but it supports more than just scalar and vector values.

Each value is an [instanced struct](https://docs.unrealengine.com/5.3/en-US/API/Plugins/StructUtils/FInstancedStruct/) (from the `StructUtils` plugin), which means it can be any struct type.

```cpp
UCLASS(MinimalAPI, BlueprintType, DisplayName="Parameter Collection")
class USunParameterCollection : public UDataAsset
{
    GENERATED_BODY()

public:
    /** Indicates the priority of this collection compared to other collections. */
    UPROPERTY(EditAnywhere, Category="Parameter Collection", meta=(InlineCategoryProperty))
    ESunParameterCollectionPriority Priority;

    /** The parameters in this collection. */
    UPROPERTY(EditAnywhere, Category="Parameter Collection", meta=(ForceInlineRow, BaseStruct="/Script/GlobalParameters.SunParameterBase", ExcludeBaseStruct, ShowOnlyInnerProperties))
    TMap<FGameplayTag, FInstancedStruct> Parameters;

    // - IsDataValid (to check for duplicate parameter names)
    // - Getter and setter functions for each parameter type using a template
}
```

Using the `BaseStruct` metadata for the `Parameters` property, I limit the parameter value to one of the following:

```cpp
// GENERATED_BODY and UPROPERTY macros omitted for brevity.
// Each Value UPROPERTY needs the `EditAnywhere` modifier.

USTRUCT() struct FSunParameterBase;
USTRUCT(DisplayName="Feature Flag") struct FSunFlagParameter : public FSunParameterBase { bool Value; };
USTRUCT(DisplayName="Scalar") struct FSunScalarParameter : public FSunParameterBase { float Value; };
USTRUCT(DisplayName="Color") struct FSunLinearColorParameter : public FSunParameterBase { FLinearColor Value; };
USTRUCT(DisplayName="Brush") struct FSunSlateBrushParameter : public FSunParameterBase { FSlateBrush Value; };
USTRUCT(DisplayName="Font") struct FSunSlateFontParameter : public FSunParameterBase { FSlateFontInfo Value; };
```

Unreal Editor has built-in detail customization for `FInstancedStruct`. When adding a parameter to a Parameter Collection, I'm able to pick any one of above structs as the value type.

<img src="/assets/images/dev-log-02-collection.png" alt="Details panel for a Parameter Collection data asset. A font parameter and a scalar parameter are visible. Pop-up menu is visible with None, Feature Flag, Color, Scalar, Brush, and Font options.">

Each collection has a `Priority` property that provides control over how multiple collections are simultaneously loaded at runtime.

```cpp
UENUM()
enum class ESunParameterCollectionPriority
{
    /* The collection represents default values. */
    Default = 0,

    /** The collection represents user preferences (including accessibility). */
    UserPreferences = 1,

    /** The collection represents a patch. */
    Patch = 2
};
```

Right now, I'm unsure about this order. I may rearrange and/or add new priority levels later.

### Runtime state
Accessible UI requires fluid layouts and that's hard to get right. There will be many, many iterations to make a widget look good in both normal and accessible modes. To make this easier, I need to preview what a UMG widget looks like with one or more accessiblity setting turned on in the widget designer. This rules out any gameplay framework objects including `UGameInstance`.

I ended up with a custom `UEngineSubsystem`. My subsystem maps each World Context's handle to a runtime collection. This means the editor and each PIE session have their own separate runtime state.

A runtime collection maintains a list of pointers to collections, sorted by priority, for each parameter.

When a parameter is queried, the collection with the highest priority is returned. A templated function in the Blueprint function library reads the value directly from the collection.

```cpp
UCLASS(MinimalAPI)
class USunGlobalParameterStatics final : public UBlueprintFunctionLibrary
{
  GENERATED_BODY()

public:
    /** Retrieves a global font parameter value or the default value if the parameter was not set. */
    UFUNCTION(BlueprintPure, Category="Global Parameters", meta=(WorldContext="WorldContextObject"))
    static GLOBALPARAMETERS_API FSlateFontInfo GetGlobalFontParameter(UObject* WorldContextObject, FGameplayTag ParameterName, FSlateFontInfo DefaultValue = FSlateFontInfo())
    {
        return GetParameter<FSunSlateFontParameter, FSlateFontInfo>(WorldContextObject, ParameterName, DefaultValue);
    }

    // GetGlobalScalarParameter, GetGlobalColorParameter, etc...

private:
    static FSunRuntimeParameterCollection* GetRuntimeCollectionFromWorldContextObject(UObject* WorldContextObject);
    static FWorldContext* GetWorldContextFromObject(UObject* WorldContextObject);

    template<typename TParameter, typename TParameterType>
    static TParameterType GetParameter(UObject* WorldContextObject, FGameplayTag ParameterName, TParameterType DefaultValue)
    {
        if (const FSunRuntimeParameterCollection* Collection = GetRuntimeCollectionFromWorldContextObject(WorldContextObject))
        {
            // GetParameterValue calls GetPtr<TParameter>() on the FInstancedStruct.
            if (const TParameter* ParameterValue = Collection->GetParameterValue<TParameter>(ParameterName))
            {
                return ParameterValue->Value;
            }
        }

        return DefaultValue;
    }
}
```

This approach makes changes to a parameter collection apply in realtime during a PIE session.

### Observing changes
Rather than querying in each tick, an `UObject` may subscribe to one or more parameters.

<img src="/assets/images/dev-log-02-observing.png" alt="Blueprint graph with Construct event linked to Add Global Parameter Observer with gameplay tags as input. Destruct event is linked to Remove Global Parameter Observer. On Global Parameter Changed event is linked to Switch On Gameplay Tag.">

`HidePin` and `DefaultToSelf` metadata modifiers are used to implicitly select the calling object as the observer.

```cpp
UFUNCTION(BlueprintCallable, Category="Global Parameters", meta=(HidePin="Observer", DefaultToSelf="Observer"))
static GLOBALPARAMETERS_API void AddGlobalParameterObserver(FGameplayTagContainer Parameters, UObject* Observer);

UFUNCTION(BlueprintCallable, Category="Global Parameters", meta=(HidePin="Observer", DefaultToSelf="Observer"))
static GLOBALPARAMETERS_API void RemoveGlobalParameterObserver(FGameplayTagContainer Parameters, UObject* Observer);
```

Following a similar approach as `UGameFeaturesSubsystem`, the observing `UObject` must implement an interface to be notified. I find this easier to work with  in a Blueprint graph compared to binding and unbinding delegates.

```cpp
class ISunGlobalParameterObserverInterface
{
    GENERATED_BODY()

public:
    /** Raised when an observed global parameter value changes. */
    UFUNCTION(BlueprintImplementableEvent, Category="Global Parameters")
    void OnGlobalParameterChanged(FGameplayTag ParameterName);
};
```

### Loading collections
There are two ways I can load and unload Parameter Collections.

When loading a save game, I use the `AddGlobalParameterCollection` and `RemoveGlobalParameterCollection` Blueprint functions.

<img src="/assets/images/dev-log-02-add-collection.png" alt="Blueprint graph of On Save Game Loaded function linked to Add Global Parameter Collection with a parameter collection as input.">

Otherwise, I use the `Add Global Parameter Collection` Game Feature action.

<img src="/assets/images/dev-log-02-overview.png" alt="Details panel for a Game Feature data asset. Add Global Parameter Collection action has a reference to a parameter collection asset.">

Using the Game Feature action is the preferred approach. When the Game Feature is activated (or deactivated), the collection is added (or removed) for each world context.

```cpp
void UGameFeatureAction_AddGlobalParameterCollection::OnGameFeatureActivating(FGameFeatureActivatingContext& Context)
{
    auto& GlobalParameterSubsystem = USunGlobalParameterSubsystem::Get();
    for (FWorldContext WorldContext : GEngine->GetWorldContexts())
    {
        if (Context.ShouldApplyToWorldContext(WorldContext))
        {
           GlobalParameterSubsystem.AddParameterCollection(WorldContext, GlobalParameterCollection);
        }
    }
}

void UGameFeatureAction_AddGlobalParameterCollection::OnGameFeatureDeactivating(FGameFeatureDeactivatingContext& Context)
{
    auto& GlobalParameterSubsystem = USunGlobalParameterSubsystem::Get();
    for (FWorldContext WorldContext : GEngine->GetWorldContexts())
    {
        if (Context.ShouldApplyToWorldContext(WorldContext))
        {
            GlobalParameterSubsystem.RemoveParameterCollection(WorldContext, GlobalParameterCollection);
        }
    }
}
```

## What's next?
The next step is to build a library of reusable widgets that have most of this wired up. Having semantic widgets like `ContentTextBlock` and `TitleTextBlock` that observe accessibility modifiers will help me make my game accessible with lesser effort.

I hope this article has helped you in one way or another. 😊

Over the past few months, I built several plugins and components I think is cool and really want to share with you all. The next dev log will be about how I used a [StateTree](https://docs.unrealengine.com/5.3/en-US/overview-of-state-tree-in-unreal-engine/) as a UI system.
