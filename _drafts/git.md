## Introduction
### What is version control?
You've been working on your game for many days, months, or even *years*, but one day—*poof*—it's gone! Maybe your game was accidentally deleted or overwritten, or maybe your storage disk became corrupted. There are many reasons but it only takes one for all the hard work you've spent on your game to be wiped out.

This is why version control systems (aka source control) are especially important when you're developing a game. Version control keeps track of all changes to your files, and makes it easy to recover any files from a previous point in time.

### Which version control system should I use?
Unreal Engine supports Perforce, Git, Subversion, and Plastic SCM. 

**Perforce** is the game industry standard but has a steep learning curve for inexperienced developers and limited free cloud hosting options. I want to write about Perforce someday but today, I'll be writing about Git. 

**Git** is the most popular among developers and many are naturally inclined to want to use it for game development too.

As for the rest, **Subversion** has generally fallen out of favor, and I am not familiar with **Plastic SCM** so I cannot comment on that one.

A version control system should have the following features to be viable for game development:
- Centralized storage for binary files. Checking out a repository should not download every revision made to a binary file.
- Ability to lock individual files from modification by other team members.

### Git and Git LFS
[Git](https://git-scm.com/) is a version control system designed for source code and other text files. It is not designed for large binary files that are frequently modified. When a Git repo is cloned, the entire history of the repo is copied which means *all versions* of each file are downloaded. Games have many large binary files that change often such as textures, meshes, audio files, levels and blueprints. Having to copy the entire history of each one of these assets is, needless to say, problematic.

[Git Large File Storage](https://www.atlassian.com/git/tutorials/git-lfs) (LFS) extends Git by replacing these large files with small text files that point to a remote location where the file actually exists. It seamlessly downloads the correct version of the file in your local copy of the repo. Git LFS also provides the ability to lock individual files so that developers may avoid merge conflicts on binary files.

**With Git LFS, Git becomes a viable choice for game development!**

### What's covered in this guide?
You'll find a step-by-step guide on setting up Unreal Engine integration with Git and Git LFS followed by info on how to store your repo in the cloud for **free** via [GitHub](https://github.com/) or [Azure Repos](https://azure.microsoft.com/en-us/products/devops/repos). You'll also find my `.gitignore` and `.gitattributes` files which have served me well in my personal Unreal projects.

## Prerequisites
In order to use Git with Unreal Engine, you must have both **Git** and **Git LFS** installed! If not, follow these steps to install it.

### Install Git for Windows
[Download Git for Windows](https://gitforwindows.org/). This is a bundle containing Git plus various tools including Git LFS which you'll need. Run the executable you just downloaded to launch setup.

The default for all settings will be fine, but make sure **Git LFS (Large File Support)** is checked on this step.

[step 3 image]

If you need some extra help, click on the button below to reveal the detailed setup walkthrough.

<details style="margin-bottom: 1em;">
<summary><span style="cursor: pointer;">Toggle setup walkthrough</span></summary>

[step 1 image]

Click **Next** to continue.

[step 2 image]

This step is where you can select where Git will be installed. The default location should be fine. Click **Next** to continue.

[step 3 image]

**IMPORTANT:** Make sure **Git LFS (Large File Support)** is checked! Everything else is optional. Click **Next** to continue.

[step 4 image]

Click **Next** to continue.

[step 5 image]

I recommend using Visual Studio Code as Git's default editor. Click **Next** to continue.

[step 6 image]

This is where you can set the default Git branch name. While the choice is entirely up to you, I suggest setting the default branch name to `main` because that's the current standard. Click **Next** to continue.

[step 7 image]

Make sure the middle choice is selected and click **Next** to continue.

[step 8 image]

The default value is fine. From now on, just keep clicking **Next** until you get to the end.

[step 9 image]

[step 10 image]

[step 11 image]

[step 12 image]

[step 13 image]

[step 14 image]

[step 15 image]

Click **Install**.

[step 16 image]

Finally! Git has been installed and configured. Click **Finish** to close setup.
      
</details>

### Verify Installation
Let's verify that both Git and Git LFS were successfully installed.

[Open the Windows Terminal (or Command Prompt)](https://www.minitool.com/news/open-command-prompt-windows-11.html) and enter `git --version` and then `git lfs --version`.

```console
C:\Users\citrus>git --version
git version 2.40.0.windows.1

C:\Users\citrus>git lfs --version
git-lfs/3.3.0 (GitHub; windows amd64; go 1.19.3; git 77deabdf)

C:\Users\citrus>
```

When you see output similar to above, then you're all set and ready for the next step! The version and other details may be different, but that's alright as long you've downloaded the latest version.

## Setup
### Create a repo
Open the folder containing your game's `.uproject` file.

[uproject-image-1]

Click on an empty space in the path at the top. Type in `cmd` and then press <kbd>Enter</kbd>. This will open a terminal window pointing to this folder.

[uproject-image-2]

Type `git init` and then press <kbd>Enter</kbd>. This will initialize a Git repo in this folder.

```console
C:\Games\MyGame>git init
Initialized empty Git repository in C:/Games/MyGame/MyGame/.git/
```

At this point, no files are tracked. A file is "tracked" when Git begins monitoring the file for changes.

We'll start adding files to be tracked soon but first, we need to create a `.gitignore` file so that only relevant files are tracked. We'll also need to create a `.gitattributes` file which will tell Git that certain file types should be handled by Git LFS instead of Git.

### Add `.gitignore`
A `.gitignore` file tells Git which files should _not_ be tracked. When you clone the repo somewhere else, only tracked files will carry over.

Unreal Engine generates many files that do not need to be tracked. Whenever these "intermediate" files are missing, Unreal Engine will regenerate them. For this reason, we want to make sure only relevant content are tracked and let Unreal Engine regenerate everything else. This significantly reduces the size of the repo.

Open your favorite text editor. Copy and paste everything below and save it as a new file named `.gitignore` in the same folder as your game's `.uproject` file (for example, `MyGame.uproject`).

```gitignore
# Miscellaneous Visual Studio files.
.vs/
.vsconfig
UpgradeLog.htm

# Miscellaneous JetBrains Rider files.
.idea/

# Ignore RiderLink Plugin as it can be reinstalled by JetBrains Rider.
Plugins/Developer/RiderLink/

# Unreal Engine will regenerate the solution file.
*.sln

# Exclude cached files.
DerivedDataCache/

# Exclude compiled files.
Binaries/
Intermediate/

# Exclude generated config files and autosaves.
Saved/

# Exclude Unreal Engine starter content because it can be added via Unreal Editor anytime.
Content/StarterContent/

```

### Add `.gitattributes`
A `.gitattributes` file instructs Git to handle certain files or directories differently. With this file, we can tell Git to hand off binary files to Git LFS.

Once again, with your favorite text editor, copy and paste everything below and save it as a new file named `.gitattributes` in the same folder as `.gitignore`.

```gitattributes
*.uasset filter=lfs diff=lfs merge=lfs -text lockable
*.umap filter=lfs diff=lfs merge=lfs -text lockable
```

### Initial commit
Now, let's create the first commit! Open a terminal in the folder in [the same way you did for `git init`](#create-a-repo).

Enter `git add .` to add all eligible files to the commit.

Let's verify that the correct files have been tracked by entering `git lfs status`. The output lists all files that will be included in the commit.

```console
C:\Games\MyGame>git add .

D:\Games\MyGame>git lfs status

Objects to be committed:

        .gitattributes (Git: 1f98718)
        .gitignore (Git: 20d5661)
        Config/DefaultEditor.ini (Git: e718c17)
        Config/DefaultEditorPerProjectUserSettings.ini (Git: 07242cd)
        Config/DefaultEngine.ini (Git: 93fda30)
        Config/DefaultGame.ini (Git: ec595ae)
        Config/DefaultInput.ini (Git: 07593e6)
        Content/Characters/MyCharacter.uasset (LFS: 014069f)
        Content/L_MyMap.umap (LFS: bb3cd09)
        MyGame.uproject (Git: 2940acb)
        Source/MyGame.Target.cs (Git: 364f3b5)
        Source/MyGame/MyGame.Build.cs (Git: bc7e337)
        Source/MyGame/MyGame.cpp (Git: 348acde)
        Source/MyGame/MyGame.h (Git: 30af2b6)
        Source/MyGame/MyGameCharacter.cpp (Git: baf5e70)
        Source/MyGame/MyGameCharacter.h (Git: a739e8d)
        Source/MyGame/MyGameGameMode.cpp (Git: 3a676c4)
        Source/MyGame/MyGameGameMode.h (Git: 3df2e75)
        Source/MyGameEditor.Target.cs (Git: 73f938c)


C:\Games\MyGame>
```

Double check the list to confirm that all of your `.uasset`, `.umap`, and source code files are included in the commit. 

In contrast with `git status`, using `git lfs status` lets us verify that binary files will be handled by LFS rather than Git. We can see from the output that these two files are handled by LFS:

```
Content/Characters/MyCharacter.uasset (LFS: 014069f)
Content/L_MyMap.umap (LFS: bb3cd09)
```

If everything looks good to you, create the initial commit with `git commit -m "Initial commit"`.

```console
C:\Games\MyGame>git commit -m "Initial commit"
[main (root-commit) 5f241bf] Initial commit
 21 files changed, 210 insertions(+)
 create mode 100644 .gitattributes
 create mode 100644 .gitignore
 create mode 100644 Config/DefaultEditor.ini
 create mode 100644 Config/DefaultEditorPerProjectUserSettings.ini
 create mode 100644 Config/DefaultEngine.ini
 create mode 100644 Config/DefaultGame.ini
 create mode 100644 Config/DefaultInput.ini
 create mode 100644 Content/Characters/MyCharacter.uasset
 create mode 100644 Content/L_MyMap.umap
 create mode 100644 MyGame.uproject
 create mode 100644 Source/MyGame.Target.cs
 create mode 100644 Source/MyGame/MyGame.Build.cs
 create mode 100644 Source/MyGame/MyGame.cpp
 create mode 100644 Source/MyGame/MyGame.h
 create mode 100644 Source/MyGame/MyGameCharacter.cpp
 create mode 100644 Source/MyGame/MyGameCharacter.h
 create mode 100644 Source/MyGame/MyGameGameMode.cpp
 create mode 100644 Source/MyGame/MyGameGameMode.h
 create mode 100644 Source/MyGameEditor.Target.cs
 
C:\Games\MyGame>
```

If you're new to Git, I recommend reading [the Git tutorial](https://git-scm.com/docs/gittutorial).

It's important to be aware that while you have versioning and change tracking active now, it's still only on your computer. Read the next section to learn about cloud hosting and how you can push your repo to the cloud for free.

## Cloud Hosting
### GitHub vs Azure Repos
There are many choices for cloud Git repo hosting but only a few support Git LFS. The only ones I'm aware of are [GitHub](https://github.com/) and [Azure Repos](https://azure.microsoft.com/en-us/products/devops/repos) (and it's no a coincidence that both are Microsoft products).

|Service|Max File Size|Total Git LFS Size|Bandwidth|
|-------|---------------------------|------------------|---------|
|GitHub|2 GB|1 GB free<br/>or<br/>$5 per 50 GB|1 GB free<br/>or<br/>$5 per 50 GB|
|Azure Repos|50 GB*|Unlimited|Unlimited|

\*The entire file must be uploaded in less than 1 hour.

### GitHub
TODO: guide

### Azure Repos
TODO: guide

## Unreal Editor and Git integration
### Activate source control in the Unreal Editor
Open your game in the Unreal Editor. At the bottom right corner, there is a button labeled **Source Control**.

[ue-source-control-2]

Click on it to reveal a menu, then click on **Connect to Source Control**. This will bring up the Source Control Login window.

[ue-source-control-3]

Change the **Provider** to **Git (beta version)**. The value for Git Path should be changed if Git was installed in a different location during Git for Windows setup.

Click on **Accept Settings** to activate Git integration.

[ue-source-control-4]

If everything went well, you'll see a confirmation message pop up.

[ue-source-control-5]

The Source Control button will also have a green checkmark next to it.

### Submitting changes
TODO
