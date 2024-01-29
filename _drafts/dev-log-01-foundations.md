# Dev Log 01: Foundations of a Game

[Cozy games](https://www.reddit.com/r/CozyGamers/)—with their relaxing and stress-relieving experience—have gained in popularity, and I am a huge fan of them. Unfortunately, I haven't found a game that feels sufficiently satisfying, inspiring me to make my own cozy life simulation game centered around two things I enjoy: trains and traveling!

In these dev logs, I share my thought process and decision-making in hopes that others find it interesting. The logs will cover both technical and game design topics.

## Why Unreal Engine?
TODO

## Chapter 1: Architecture
The architecture of a game makes a lasting impact in every way, yet is very difficult to change later on.

A well-designed architecture is the glue that holds any software, especially games, together. A poorly designed architecture not only makes it harder to implement features, but also manifests in visible ways like longer loading times and a less polished player experience.

That's why **the first thing I did was design my game's structure** based on lessons learned from [Lyra](https://docs.unrealengine.com/5.3/en-US/lyra-sample-game-in-unreal-engine/) and my professional experience as a software engineer.

### Goals
Let's begin with three overarching set of goals I hope to achieve with my architecture:

#### 1. Modular: Develop features as self-contained plugins
With the aptly named [Game Features](https://docs.unrealengine.com/5.3/en-US/game-features-and-modular-gameplay-in-unreal-engine/) plugin, features exist as self-contained plugins. Deactivating any game feature will not prevent the game from running.

Of course, there has to be some mechanism for one feature to interact with another. Game features pick from a suite of "system" libraries for interoperability. More on this in [Chapter 3 - System Libraries](#chapter-3-system-libraries).

#### 2. Designer-focused: Create building blocks in C++ and assemble features in Blueprints
TODO

#### 3. Data-driven: Configure features with data assets and data sources
TODO

## Chapter 2: Initial Project Structure
The game's project starts with the following hierarchy and a couple of assets:

```
🕹️Sunshine
 ├── 📂Config
 │   ├── ...
 │   └── DefaultSunshine.ini
 ├── 📂Content
 │   ├── 📂Developers
 │   │   └── 📂Matt
 │   │       └── (experimental assets)
 │   ├── 📂Movies
 │   │   └── UE_Brand.mp4
 │   ├── 📂Splash
 │   │   └── Splash.bmp
 │   └── 📂Legal
 │       └── ThirdPartyNotices.txt
 ├── 📂Plugins
 │   ├── 📂GameFeatures
 │   │   └── (game feature plugins)
 │   ├── 📂System
 │   │   └── (system libraries)
 │   └── 📂UI
 │       └── (generic UI components)
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

As I've mentioned before—even as a solo developer—being able to pivot between separate roles (the designer and engineer roles here) and maintain a system that supports this flexibility is important.

#### Content

##### Developers folder
To support experimentation and development, I strongly encourage everyone to [enable the Developers folder](https://docs.unrealengine.com/5.3/en-US/developers-folder-in-unreal-engine/).

Having my own sandbox folder allows me to freely create Blueprints and assets without worrying about cleanup afterwards. Additionally, I've excluded this folder from cooked builds in Packaging Settings to avoid accidentally including test assets in the distributed game.

##### Movies
While entirely optional, I prefer starting with at least one startup movie to observe the transition from startup into the first map, i.e., title screen. The [animated Unreal Engine logo](https://www.unrealengine.com/en-US/branding) is used as a placeholder.

##### Splash
Optional as well, a temporary splash image can serve as inspiration. I've created a simple placeholder splash with my project's name on it.

##### Legal
Software licenses must be taken seriously, even as an indie developer.

All applicable third-party licenses are tracked in `ThirdPartyNotices.txt`. When using an open-source plugin I haven't created, I add the plugin's name and its license (usually the MIT License) to this file.

I configured *Additional Non-Asset Directories To Copy* in Packaging Settings to automatically include this folder for distribution.

#### /Plugins/
TODO

#### /Source/
TODO

## Chapter 3: System Libraries
TODO
