# Dev Log 01: Untitled

[Cozy games](https://www.reddit.com/r/CozyGamers/)—with their relaxing and stress-relieving experience—have gained in popularity, and I am a huge fan of them. Unfortunately, I haven't found a game that feels sufficiently satisfying, inspiring me to make my own cozy life simulation game centered around two things I enjoy: trains and traveling!

In these dev logs, **I share my thought process and decision-making in hopes that others find it interesting**. The logs will cover both technical and game design topics.

## Designing the game's architecture
The architecture of a game makes a lasting impact in every way, yet is very difficult to change later on. **A well-designed architecture is the glue that holds any software—especially games—together**.

A poorly designed architecture not only makes it harder to implement features, but also manifests in visible ways like longer loading times and a less polished player experience.

But wait, this goes both ways! A friend of mine, an engineer at 343 Industries, warned me that **"how well [the] architecture is defined lends into how hard is to adapt more/new features."** An architecture that is too opinionated will have the same negative impact to a game as an architecture that is too flexible.

How do I know whether my architecture is too rigid or too flexible? The most one can do is learn from prior experience. In my capacity as a software engineer at Google (and formerly Microsoft), I am fortunate to have the opportunity to learn how good software systems are designed. Although I did not work on games, general software engineering principles and best practices absolutely do apply to video games.

**I will make mistakes as I develop my game**. My architecture probably won't be the "correct" or "best" way of doing things. All I can do is continue to learn from my mistakes as well as learn from others. A game developer must make a lifetime commitment to learning.

Let's begin with a set of three overarching goals I hope to achieve with my architecture:

1. **Modular**: Develop features as self-contained plugins.
2. **Designer-focused**: Create building blocks in C++ and assemble features in Blueprints.
3. **Data-driven**: Configure features with data assets, data tables, and registries.

### Develop features as self-contained plugins
With the aptly named [Game Features](https://docs.unrealengine.com/5.3/en-US/game-features-and-modular-gameplay-in-unreal-engine/) plugin, features exist as self-contained plugins. Deactivating any game feature will not prevent the game from running.

Of course, there has to be some mechanism for one feature to interact with another. Game features rely on a suite of "system" libraries for interoperability. More on this in [System Libraries](#system-libraries).

Now, let's clarify what I mean by a "feature". In my view, a feature is any element that adds to the core sandbox experience. **The sandbox is the minimum viable player experience**, and features are extensions built upon this sandbox.

In other words, features provide *content* to the game. Individual features will be bundled together into a *Game Feature* plugin, serving as a content pack.

Why does it matter how it's defined? With a clear definition, I'm able to determine where an asset or class belongs, whether it goes into the sandbox or added as a Game Feature plugin.

These *are* features:
* Characters (except archetypes)
* Items (e.g., clothes, furniture, consumables)
* Maps
* Quests
* [Gameplay abilities](https://docs.unrealengine.com/5.3/en-US/gameplay-ability-system-for-unreal-engine/)
* Title Menu & Lobby*

\*The title menu and lobby are not necessary for the sandbox itself to function, and therefore can be isolated as a Game Feature plugin. For example, a demo build of the game may launch directly into the sandbox.

And these are *not* features:

* I/O systems (e.g., input, game saves, UI, haptics)
* Scalability, audio, and accessibility settings
* `AGameState` and `APlayerState`
* Character archetypes
* Inventory system*

*Items cannot exist without an inventory system, therefore the inventory system is a game mechanic within the sandbox, and not a feature.

### Create building blocks in C++ and assemble features in Blueprints
**Systems, APIs, and tools are written in C++ and then called in Blueprints**. Designers use these as building blocks to implement a feature. This is the intended way of using Unreal Engine. If I tried to create the whole game in either C++ or Blueprints alone, I would end up fighting against the engine.

As a solo developer, I will take on multiple roles: producer, writer, designer, programmer, and more. I write code in my role as a programmer that I can use in my role as a technical designer.

### Define content with data assets, data tables, and registries
!!! TODO: Rewrite this section !!!

```
Suppose there is a flower pot that can be picked up by the player. The incorrect approach would be to create an actor named `AFlowerPot`, subclassed from `APickupableObject`, which is then subclassed from `AWorldObject`.

Instead, I will create an archetype for each actor type and use data assets to set each actor's behavior and appearance.

Using the above example, there is an `AWorldObject` actor class and a corresponding data asset class, `UWorldObjectData`. The actor class has a property that points to a data asset of this type. This data asset contains properties that control the appearance and behavior of the world object, such as a flag indicating whether it can be picked up by the player, the physical space it occupies, and how it should be rendered.

Data tables, and in some cases, [data registries](https://docs.unrealengine.com/5.3/en-US/data-registries-in-unreal-engine/), are used to register features for use within the sandbox.
```

## Creating the initial project
My project is created with the following hierarchy and a few assets:

```
🕹️Sunshine
 ├── 📂Config
 │   ├── ...
 │   └── DefaultSunshine.ini
 ├── 📂Content
 │   ├── 📂Developers
 │   │   └── 📂Matt
 │   ├── 📂Maps
 │   │   └── L_Default.umap
 │   ├── 📂Movies
 │   │   └── UnrealEngine.mp4
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

[JetBrians Rider](https://www.jetbrains.com/rider/) is my IDE and it works directly with `.uproject`. That's why I don't have a Visual Studio `.sln` solution file.

### EditorConfig
Both Visual Studio and JetBrains Rider support [EditorConfig](https://editorconfig.org/). **By placing an `.editorconfig` file in the project's root folder, I enforce a consistent code style throughout the entire project**.

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

Feel free to copy my [.editorconfig file](https://gist.github.com/the-unrealist/861fdc90c0e13b88c46be68a6418b80d). Note that most of the rules were automatically added by Rider so I am unsure if they work in Visual Studio.

### Git
Although Perforce is the industry standard, I'm very conscious of my budget as a solo developer and don't want to pay for cloud hosting if I can avoid it. This is why I am pleased to learn that **[Azure DevOps](https://azure.microsoft.com/en-us/products/devops) offers free unlimited Git Large File Storage (LFS) hosting**! For this reason, I use Git and Git LFS as my game's version control system.

To enable Git LFS, I created a `.gitattributes` file targeting `.uasset`, `.umap`, and other binary files. These assets will use binary-compatible Git LFS instead of text-based Git for diffs. This is absolutely essential for game development.

```gitignore
# Unreal Assets
*.uasset filter=lfs diff=lfs merge=lfs -text lockable
*.umap filter=lfs diff=lfs merge=lfs -text lockable

# Non-UFS Assets
*.mp4 filter=lfs diff=lfs merge=lfs -text lockable
*.bmp filter=lfs diff=lfs merge=lfs -text lockable
```

It's good practice to run `git lfs status` before each commit to ensure all binary files are properly handled by Git LFS. 

It's not feasible to merge two binary files so file locking is the standard operating procedure when working with assets. Each rule in my `.gitattributes` includes the `lockable` flag which makes binary files readonly by default. To make changes to a binary file, I must try to lock it first. Unreal Editor has built-in support for file locking. I get prompted to lock a file when I attempt to save changes to an asset.

Now, we don't want to check nonessential files into version control. I created a `.gitignore` file to exclude auto-generated files. It's a habit of mine to craft my own exclusion list from scratch. In this case, there is no need to add Visual Studio files to the list because I use JetBrains Rider, so files like `.sln` do not appear in my workspace at all.

```gitignore
# Miscellaneous JetBrains Rider files.
.idea/
*.DotSettings.user

# Ignore RiderLink Plugin since it's automatically reinstalled by JetBrains Rider.
Plugins/Developer/RiderLink/

# Exclude compiled files.
Binaries/
Intermediate/

# Exclude generated config files and autosaves.
Saved/

# Exclude other generated Unreal Engine directories.
DerivedDataCache/
Build/
```

### Config
`DefaultSunshine.ini` stores the project-level config for all of my [custom building blocks and game features](#2-designer-focused-create-building-blocks-in-c-and-assemble-features-in-blueprints). This assists in my mission to be designer-focused by clearly delineating game-related config for designers to modify. `DefaultGame.ini` is still used for configuring most Unreal Engine features.

By default, Unreal will not cook any custom configuration file. `DefaultSunshine.ini` must be allowlisted in `DefaultGame.ini`.

```ini
[Staging]
+AllowedConfigFiles=Sunshine/Config/DefaultSunshine.ini
```

### Content

#### Developers
To support experimentation and development, I enabled the [Developers folder](https://docs.unrealengine.com/5.3/en-US/developers-folder-in-unreal-engine/).

Having my own sandbox folder allows me to freely create Blueprints and assets without worrying about cleanup afterwards. Additionally, I've excluded this folder from cooked builds in Packaging Settings to avoid accidentally including test assets in the distributed game.

#### Maps
This folder contains internal system maps such as an empty `L_Default` map that is used as the fallback map. Maps for the game belong in Game Feature plugins.

#### Movies
While entirely optional, I prefer starting with at least one startup movie to observe the transition from startup into the first map, i.e., the title screen. The [animated Unreal Engine logo](https://www.unrealengine.com/en-US/branding) is used as a placeholder.

#### Splash
Optional as well, a temporary splash image can serve as inspiration. I've created a simple placeholder splash with my project's name on it.

#### Sunshine
Assets for my game's sandbox are located in this folder.

#### Legal
Software licenses must be taken seriously, even as an indie developer.

All applicable third-party licenses are tracked in `ThirdPartyNotices.txt`. When using an open-source plugin I haven't created, I add the plugin's name and its license (usually the MIT License) to this file.

I configured *Additional Non-Asset Directories To Copy* in Packaging Settings to automatically include this folder for distribution.

### Plugins
There are three categories under which I'll organize my plugins:

#### Game features
Game Feature plugins provide "content" to my game. Since the sandbox must be built first before features can be added, this folder remains empty for now.

#### System
[System libraries](#system-libraries) provide APIs to the sandbox. Game Feature plugins extend and interact with the sandbox using one or more system libraries.

#### UI
Plugins that provide reusable Slate and UMG widgets for the game are placed under this folder. This makes it easier to distribute useful widgets for use in other games or on the marketplace.

### Source
This folder holds the source code for the sandbox.

I created two game modules: `SunshineGame` and `SunshineEditor`. `SunshineGame` contains all the code needed to implement the sandbox, and `SunshineEditor` contains editor-only tooling to make development easier.

## Defining system libraries
This is a suite of libraries that provide an interface to various components of the sandbox. The sandbox exists as a black box with controlled access via system libraries.

An interface in this context means a designer-friendly API to the sandbox for game features. They are typically implemented as a Blueprint-callable function library or possibly [custom Blueprint nodes](https://unrealist.org/custom-blueprint-nodes/).

Many features require access only to specific parts of the sandbox, and some interfaces may even require a different module load phase. For these reasons, I've split the sandbox interface into this (non-exhaustive) set of discrete plugins:

|System Plugin Name|Provides access to...|
|-------------------|----------------------|
|Event Stream|Pub/sub system for events and messaging|
|Observability|Logging and telemetry|
|Input|Input contexts and actions|
|HUD|Player UI while in-game|
|Menu|Modals and menus|
|Settings|Settings system|
|Conversations|NPC dialogue system|
|Inventory|Inventory system|
|Abilities|Gameplay ability system|
|Persistent State|Save game system|
|Actors|Archetypes for modular actors|

More system libraries will be added as the need arises.

## Preparing for a multilingual audience
It would be fantastic to share my game with the whole world, but for this to happen, my game must first be translated into different languages.

It would be a mistake to think about localization after the fact. Localization in my game was initialized right from the beginning before I write the first line of text.

I performed these steps to set my project up for localization:

1. Open the Localization Dashboard.
2. Select the *Game* target.
3. Check *Gather from Packages*
4. Add `Sunshine/Content/*` to *Include Path Wildcards*.
5. Verify `.uasset` and `.umap` are in *File Extensions*.

Over time I will keep pressing the *Gather Text* button to scan for all localizable text in my game. To be honest I haven't really looked into how translations will work here. However, I see buttons for exporting and importing translations, so I believe this is the right step.

This tool generates several text and binary files. I included the following in my `.gitattributes` to target these new binary files:

```gitignore
*.locmeta filter=lfs diff=lfs merge=lfs -text lockable
*.locres filter=lfs diff=lfs merge=lfs -text lockable
```

Some files are encoded as UTF-16 with a byte order mark (BOM). Git mistakenly handles these files as binary for diffs. To fix this, I also added the following to my `.gitattributes`:

```gitignore
*.archive diff working-tree-encoding=UTF-16LE-BOM eol=CRLF
*.manifest diff working-tree-encoding=UTF-16LE-BOM eol=CRLF
```

## Verifying packages
Each time before I commit anything to my git repo, I package a shipping build to catch packaging errors early. I do *not* want to find out something's seriously wrong with my project when I'm deep into the development cycle.

Some day, I would like to set up continuous integration and delivery (CI/CD) using the [Unreal Automation Tool](https://docs.unrealengine.com/5.3/en-US/unreal-automation-tool-for-unreal-engine/) to automate this process.

## What's next?
I'm glad you've made it this far!

Initial project setup takes a lot of time, which is why I'm happy to share my project with you as a template. Feel free to [download my project template](https://github.com/TODO) and use it to start making your game!

Further dev logs will be written as I make progress in my game in the coming months. The next one will very likely be about the custom UI widgets I created recently.

I will appreciate any and all feedback from you! Please reach out in the comments below (requires a GitHub account).
