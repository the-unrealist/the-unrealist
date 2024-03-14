---
tags: 
  - unreal
title: "Dev Log 01 — Ten Essential Steps When Starting an Unreal Engine Project"
categories:
  - Dev Logs
---
<img src="https://img.shields.io/badge/Unreal%20Engine-5.3-informational" alt="Written for Unreal Engine 5.3"> <img src="https://img.shields.io/badge/-Dev%20Log-red" alt="Dev Log">

I am a huge fan of [cozy games](https://www.reddit.com/r/CozyGamers/) because they are relaxing and stress-relieving, which is what I want after a long day at work. Unfortunately, I haven't found a game that feels sufficiently satisfying yet, inspiring me to make my own cozy life simulation game centered around two things I enjoy: trains and traveling!

In these dev logs, **I share my thought process and decision-making in hopes that others find it interesting**. The logs will cover both technical and game design topics.

Let's begin with the ten essential steps I always take when creating a new Unreal Engine project:

## 1. Think about the game's architecture
The architecture of a game makes a lasting impact in every way, yet is very difficult to change later on. **A well-designed architecture is the glue that holds any software—especially games—together**.

Poorly designed architecture not only makes it harder to implement features but also manifests in visible ways, such as longer loading times and a less polished player experience. This goes the other way; an architecture that is too rigid will have the same negative impact on a game.

Now, how do I know if my architecture is *just right*? The best one can do is learn from prior experience. In my role as a software engineer, I've been fortunate to have the opportunity to dive deep into well-designed software systems. Although I did not work on games, general software engineering principles and best practices absolutely do apply to video games.

**I will make mistakes as I develop my game**. My architecture may not represent the "best" way of doing things. All I can do is learn from my mistakes as well as learn from others. A game developer must make a lifetime commitment to learning.

Let's start with three overarching goals I hope to achieve with my architecture:

1. **Modular**—Organize content into self-contained plugins.
2. **Designer-focused**—Develop building blocks in C++ and implement features in Blueprints.
3. **Data-driven**—Create content with data assets, data tables, and registries.

### Organize content into self-contained plugins
With the [Game Features](https://docs.unrealengine.com/5.3/en-US/game-features-and-modular-gameplay-in-unreal-engine/) plugin, content is organized into self-contained plugins. Deactivating any Game Feature plugin must not hinder the game from running.

Now, let's define what I mean by "content". In my view, content is anything that adds to the core sandbox experience. Here, **the sandbox is the minimum viable player experience**, and content is an extension built upon this sandbox. Individual pieces of content are bundled together as a Game Feature plugin, serving as a content pack.

Why does the definition matter? With a clear definition, I can easily determine whether an asset or class belongs in the sandbox or should be added as a Game Feature plugin.

These *are* content:
* Characters (except archetypes)
* Items (e.g., clothes, furniture, consumables)
* Maps
* Quests
* [Gameplay abilities](https://docs.unrealengine.com/5.3/en-US/gameplay-ability-system-for-unreal-engine/)
* Title Menu & Lobby*

\*The title menu and lobby are not necessary for the sandbox itself to function, and therefore can be isolated as a Game Feature plugin. For example, a demo build of the game may launch directly into the sandbox.

And these are *not* content:

* I/O systems (e.g., input, game saves, UI, haptics)
* `AGameState` and `APlayerState`
* Character archetypes
* Inventory system*

*The inventory system is a core game mechanic within the sandbox, and most content will have a dependency on the inventory system. For this reason, it is not content.

### Develop building blocks in C++ and implement features in Blueprints
**Systems, APIs, and tools are coded in C++ and then called from Blueprint graphs**. These serve as building blocks that designers use to implement features. When a Blueprint graph becomes overly complex, then parts of it will be broken down and refactored into C++.

My understanding is that this is the intended way of using Unreal Engine. Developing the entire game solely in either C++ or Blueprints would be needlessly challenging, so I am using both.

### Create content with data assets, data tables, and registries
I prefer composition over inheritance for content.

For actors, this means creating an archetype actor with a property that points to a data asset which defines the appearance and behavior of the actor.

@startmermaid
flowchart TB
    archetype["ASunshineCharacter (AActor)"]
    npc1["NPC 1 (UPrimaryDataAsset)"]
    npc2["NPC 2 (UPrimaryDataAsset)"]
    archetype--Data-->npc1
    archetype--Data-->npc2
@endmermaid

**This completely decouples content from the sandbox**, allowing me to make significant changes to actor archetypes and other parts of the sandbox without potentially invalidating hundreds of downstream assets. Actor components attached to the archetype bring it to life using data from the data asset.

Data tables and [data registries](https://docs.unrealengine.com/5.3/en-US/data-registries-in-unreal-engine/) are also useful for the same reason.

## 2. Create the initial project
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

`Sunshine` is the working title of my project. It's not the final name of my game. I need to pick (and stick to) a working title so that I can add a prefix to all of my C++ types, e.g., `USunAssetManager`. This prevents conflicting with engine types.

By the way, I use [JetBrians Rider](https://www.jetbrains.com/rider/) as my IDE, and it directly works with `.uproject`. That's why I don't have a Visual Studio `.sln` solution file.

## 3. Add initial assets

### Maps
This folder contains internal system maps such as an empty `L_Default` map that is used as the fallback map. Maps for the game are content and belong in Game Feature plugins.

### Movies
While entirely optional, I prefer starting with at least one startup movie to observe the transition from startup into the first map, i.e., the title screen. The [animated Unreal Engine logo](https://www.unrealengine.com/en-US/branding) is used as a placeholder.

### Splash
Optional as well, a temporary splash image can serve as inspiration. I've created a simple placeholder splash with my project's name on it.

### Sunshine
Assets for my game's sandbox are located in this folder.

### Legal
Software licenses must be taken seriously, even as an indie developer.

All applicable third-party licenses are tracked in `ThirdPartyNotices.txt`. When using an open-source plugin or asset, I add their license to this file.

I configured *Additional Non-Asset Directories To Copy* in Packaging Settings to automatically include this folder for distribution.

## 4. Enable the Developers folder
To support experimentation and development, I enabled the [Developers folder](https://docs.unrealengine.com/5.3/en-US/developers-folder-in-unreal-engine/).

Having my own sandbox folder allows me to freely create Blueprints and assets without worrying about cleanup afterwards. Additionally, I've excluded this folder from cooked builds in Packaging Settings to avoid accidentally including test assets in the distributed game.

## 5. Make an EditorConfig
Both Visual Studio and JetBrains Rider support [EditorConfig](https://editorconfig.org/). By placing an `.editorconfig` file in the project's root folder, I enforce a consistent code style throughout the entire project.

I recommend having these global rules to ensure all files are encoded and formatted the same way:

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

Feel free to grab a copy of my [.editorconfig file](https://gist.github.com/the-unrealist/861fdc90c0e13b88c46be68a6418b80d). Just bear in mind that most of these rules were added by Rider, so it's unclear to me whether Visual Studio understands them.

## 6. Create game and editor modules in C++
I created two game modules specifically for the sandbox: `SunshineGame` and `SunshineEditor`. `SunshineGame` contains all the code needed to implement the sandbox, and `SunshineEditor` contains editor-only tooling.

## 7. Setup source control
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

## 8. Create a project config
`DefaultSunshine.ini` stores the project-level config for all of my [building blocks and game features](#develop-building-blocks-in-c-and-implement-features-in-blueprints). By adding `Config=Sunshine` to the `UCLASS` macro, config properties will be written to this file.

Having a separate config file helps clearly delineate game-related config for designers to modify. `DefaultGame.ini` is still used for configuring most Unreal Engine features.

**By default, Unreal will not cook any custom configuration file**. `DefaultSunshine.ini` must be allowlisted in `DefaultGame.ini`.

```ini
[Staging]
+AllowedConfigFiles=Sunshine/Config/DefaultSunshine.ini
```

## 9. Prepare for a multilingual audience
Sharing my game with the whole world would be fantastic, but for this to happen, the game must first be translated into different languages.

It would be a mistake to think about localization as an afterthought. In my project, localization was set up right from the beginning, even before writing the first line of text.

To set my project up for localization, I followed these steps:

1. Open the Localization Dashboard.
2. Select the *Game* target.
3. Check *Gather from Packages*
4. Add `Sunshine/Content/*` to *Include Path Wildcards*.
5. Verify `.uasset` and `.umap` are in *File Extensions*.

As time goes on, I'll consistently press the *Gather Text* button to scan for all localizable text in my game. To be honest, I haven't really looked into how translations will work here. Nonetheless, I see buttons for exporting and importing translations, so I believe this is the right step.

The Localization Dashboard generates several files for each culture. I added `.locmeta` and `.locres` file extensions to my `.gitignore` because these are compiled from localization source files, so I believe they don't need to be tracked by git.

```gitignore
# Exclude compiled localization files.
*.locmeta
*.locres
```

Some of the generated text files are encoded as UTF-16 with a byte order mark (BOM). Git mistakenly treats these files as binary for diffs. To fix this, I added the following to my `.gitattributes`:

```gitignore
# Correctly handle generated localization file encoding.
*.archive diff working-tree-encoding=UTF-16LE-BOM eol=CRLF
*.manifest diff working-tree-encoding=UTF-16LE-BOM eol=CRLF
```

## 10. Package a shipping build
Before each commit to my git repo, I package a shipping build to catch packaging errors early on. I do *not* want to find out something's seriously wrong with my project when it's deep into the development cycle.

Some day, I would like to set up continuous integration and delivery (CI/CD) using the [Unreal Automation Tool](https://docs.unrealengine.com/5.3/en-US/unreal-automation-tool-for-unreal-engine/) to automate this process.

## What's next?
Stay tuned for more dev logs as I make progress on my game in the coming months.

Please don't hesitate to drop your feedback in the comments below (requires a GitHub account).
