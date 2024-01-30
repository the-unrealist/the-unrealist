# Dev Log 01: Building the foundation of my game

[Cozy games](https://www.reddit.com/r/CozyGamers/)—with their relaxing and stress-relieving experience—have gained in popularity, and I am a huge fan of them. Unfortunately, I haven't found a game that feels sufficiently satisfying, inspiring me to make my own cozy life simulation game centered around two things I enjoy: trains and traveling!

In these dev logs, I share my thought process and decision-making in hopes that others find it interesting. The logs will cover both technical and game design topics.

**My design decisions are subjective**. I make no claim that my approach is the "correct" or "best" way of doing things. I am sharing what works for me and my game.
 
## Architecture
The architecture of a game makes a lasting impact in every way, yet is very difficult to change later on.

A well-designed architecture is the glue that holds any software, especially games, together. A poorly designed architecture not only makes it harder to implement features, but also manifests in visible ways like longer loading times and a less polished player experience.

That's why **the first thing I did was design my game's structure** based on lessons learned from [Lyra](https://docs.unrealengine.com/5.3/en-US/lyra-sample-game-in-unreal-engine/) and my professional experience as a software engineer.

### Goals
Let's begin with a set of three overarching goals I hope to achieve with my architecture:

1. **Modular**: Develop features as self-contained plugins.
2. **Designer-focused**: Create building blocks in C++ and assemble features in Blueprints.
3. **Data-driven**: Configure features with data assets, data tables, and registries.

#### 1. Modular: Develop features as self-contained plugins
With the aptly named [Game Features](https://docs.unrealengine.com/5.3/en-US/game-features-and-modular-gameplay-in-unreal-engine/) plugin, features exist as self-contained plugins. Deactivating any game feature will not prevent the game from running.

Of course, there has to be some mechanism for one feature to interact with another. Game features rely on a suite of "system" libraries for interoperability. More on this in [System Libraries](#system-libraries).

Now, let's clarify what I mean by a "feature". In my view, a feature is any element that adds to the core sandbox experience. The **sandbox represents the minimum viable player experience**, and features are extensions built upon this sandbox. In other words, features provide *content* to the game.

##### What are NOT features?
* Input system
* UI framework
* Scalability, audio, and accessibility settings
* Game and player states
* Character archetypes
* Inventory system

All of the above are critical to the sandbox experience, but may be extended by features.

##### What are features?
* Characters
* Items
* Maps
* Dialogue
* [Gameplay abilities](https://docs.unrealengine.com/5.3/en-US/gameplay-ability-system-for-unreal-engine/)
* Title Menu & Lobby*

\*This one was tricky. I've thought about this for a while and have decided that the title menu and lobby are features. They are the frontend to the sandbox. They are not necessary for the sandbox itself to function.

Individual features will be bundled together into a *Game Feature* plugin, effectively serving as a content pack.

#### 2. Designer-focused: Create building blocks in C++ and assemble features in Blueprints
Systems, frameworks, and tools are written in C++ and then exposed to designers via Blueprints. Designers use these as building blocks to implement a feature. This is the intended way of using Unreal Engine. Otherwise, I would be fighting against the engine if I tried to create the whole game in either C++ or Blueprints alone.

As a solo developer, I will take on multiple roles: producer, writer, designer, programmer, and more. I write code in my role as a programmer that I can use in my role as a technical designer.

The sandbox in my game will be mostly using C++ and the features will be using Blueprints.

#### 3. Data-driven: Configure features with data assets, data tables, and registries
Suppose there is a flower pot that can be picked up by the player. The incorrect approach would be to create an actor named `AFlowerPot`, subclassed from `APickupableObject`, which is then subclassed from `AWorldObject`.

Instead, I will create an archetype for each actor type and use data assets to set each actor's behavior and appearance.

Using the above example, there is an `AWorldObject` actor class and a corresponding data asset class, `UWorldObjectData`. The actor class has a property that points to a data asset of this type. This data asset contains properties that control the appearance and behavior of the world object, such as a flag indicating whether it can be picked up by the player, the physical space it occupies, and how it should be rendered.

Data tables, and in some cases, [data registries](https://docs.unrealengine.com/5.3/en-US/data-registries-in-unreal-engine/), are used to register features for use within the sandbox.

## Initial Project Structure
The game's project starts with the following hierarchy and a couple of assets:

```
🕹️Sunshine
 ├── 📂Config
 │   ├── ...
 │   └── DefaultSunshine.ini
 ├── 📂Content
 │   ├── 📂Developers
 │   │   └── 📂Matt
 │   ├── 📂Movies
 │   │   └── UE_Brand.mp4
 │   ├── 📂Splash
 │   │   └── Splash.bmp
 │   ├── 📂Sunshine
 │   └── 📂Legal
 │       └── ThirdPartyNotices.txt
 ├── 📂Plugins
 │   ├── 📂GameFeatures
 │   ├── 📂System
 │   └── 📂UI
 ├── 📂Source
 ├── .editorconfig
 ├── .gitattributes
 ├── .gitignore
 └── Sunshine.uproject
```

`Sunshine` is my project name. Most custom systems and classes will have `Sunshine` as the prefix, e.g., `USunshineAssetManager`. It's not the final name of my game.

#### EditorConfig
Both Visual Studio and JetBrains Rider (which I use) support [EditorConfig](https://editorconfig.org/). By placing an `.editorconfig` file in the project's root folder, I enforce a consistent code style throughout the entire project.

I recommend having the following global ruleset to ensure all files are encoded and formatted the same way.

```editorconfig
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

Feel free to copy my [.editorconfig file](#todo). Note that some of the rules were automatically added by Rider so I am unsure if they work in Visual Studio.

#### Git
Although Perforce is the industry standard, I'm very conscious of my budget as a solo developer and don't want to pay for cloud hosting if I can avoid it. This is why I am pleased to learn that [Azure DevOps](https://azure.microsoft.com/en-us/products/devops) offers free unlimited Git Large File Storage (LFS) hosting. For this reason, I use Git and Git LFS as my game's version control system.

To enable Git LFS, I created a `.gitattributes` file targeting all `.uasset` and `.umap` files. These assets will use binary-friendly Git LFS instead of text-based Git for diffs and merges.

```gitignore
*.uasset filter=lfs diff=lfs merge=lfs -text lockable
*.umap filter=lfs diff=lfs merge=lfs -text lockable
```

I expect to include more asset file extensions as I encounter them later on.

Now, we don't want to check nonessential files into version control. I created a `.gitignore` file to exclude auto-generated files.

```gitignore
# Miscellaneous Visual Studio files.
.vs/
.vsconfig

# Miscellaneous JetBrains Rider files.
.idea/
*.DotSettings.user

# Ignore RiderLink Plugin since it's automatically reinstalled by JetBrains Rider.
Plugins/Developer/RiderLink/

# Unreal Engine will regenerate the solution file.
*.sln

# Exclude compiled files.
Binaries/
Intermediate/

# Exclude generated config files and autosaves.
Saved/

# Exclude other generated Unreal Engine directories.
DerivedDataCache/
Build/
```

#### Config
`DefaultSunshine.ini` stores the project-level config for all of my [custom building blocks and game features](#2-designer-focused-create-building-blocks-in-c-and-assemble-features-in-blueprints). This assists in my mission to be designer-focused by clearly delineating game-related config for designers to modify. `DefaultGame.ini` is still used for configuring most Unreal Engine features.

#### Content

##### Developers folder
To support experimentation and development, I enabled the [Developers folder](https://docs.unrealengine.com/5.3/en-US/developers-folder-in-unreal-engine/).

Having my own sandbox folder allows me to freely create Blueprints and assets without worrying about cleanup afterwards. Additionally, I've excluded this folder from cooked builds in Packaging Settings to avoid accidentally including test assets in the distributed game.

##### Movies
While entirely optional, I prefer starting with at least one startup movie to observe the transition from startup into the first map, i.e., title screen. The [animated Unreal Engine logo](https://www.unrealengine.com/en-US/branding) is used as a placeholder.

##### Splash
Optional as well, a temporary splash image can serve as inspiration. I've created a simple placeholder splash with my project's name on it.

##### Sunshine
Content specific to the sandbox will be located in this folder. Assets should be grouped together in directories based on purpose—not their type.

##### Legal
Software licenses must be taken seriously, even as an indie developer.

All applicable third-party licenses are tracked in `ThirdPartyNotices.txt`. When using an open-source plugin I haven't created, I add the plugin's name and its license (usually the MIT License) to this file.

I configured *Additional Non-Asset Directories To Copy* in Packaging Settings to automatically include this folder for distribution.

#### Plugins
Most of the game will be implemented as plugins under three categories:

##### Game Features
Game Feature plugins provide "content" to my game. Since the sandbox must be built first before features can be added, this folder remains empty for now.

##### System
Game Feature plugins will extend and interact with the sandbox by adding one or more of these [system libraries](#system-libraries) in this folder as dependencies.

##### UI
Plugins that provide reusable Slate and UMG widgets for the game are placed under this folder. This makes it easier to distribute useful widgets for use in other games or on the marketplace.

#### Source
This folder holds the source code for the sandbox, encompassing singletons like my subclasses of `UAssetManager` and `UGameInstance`, as well as gameplay framework actors such as `AGameMode`, `AGameState`, and `APlayerState`. The system libraries are also bootstrapped here so that features have access to the sandbox via designer-friendly interfaces.

## System Libraries
System libraries define interfaces to various components of the sandbox.

An interface in this context means a designer-friendly API to the sandbox for game features. They are typically implemented as a Blueprint-callable function or possibly [a custom `K2Node`](https://unrealist.org/custom-blueprint-nodes/).

Many features require access only to specific parts of the sandbox, and some interfaces may even require a different module load phase. For these reasons, I've split the sandbox interface into this (non-exhaustive) set of discrete plugins:

|System Plugin Name|Purpose|
|-------------------|----------------------|
|Event Stream|Pub/sub system for events and messaging.|
|Observability|Logging and telemetry.|
|Input|Register input contexts and actions.|
|HUD|Add widgets to the player's HUD.|
|Menu|Display modals and menus.|
|Settings|Register feature-specific settings.|
|Conversations|Trigger dialogue with characters.|
|Inventory|Access to the inventory system.|
|Abilities|Register gameplay abilities.|
|Persistent State|Access the save game system.|
|Actors|Archetypes for modular actors.|

More system libraries are added as the need arises.

## What's next?
I'm glad you've made it this far! Further dev logs will be written as I make progress in my game in the coming months. If you have any comments or questions, please reach out in the comments below.
