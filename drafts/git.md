1. [Introduction](#introduction)
   - [What is version control?](#what-is-version-control)
   - [Which version control system should I use?](#which-version-control-system-should-i-use)
   - [Git and Git LFS](#git-and-git-lfs)
   - [What's covered in this guide?](#whats-covered-in-this-guide)
2. [Prerequisites](#prerequisites)
   - [Install Git for Windows](#install-git-for-windows)
   - [Verify Installation](#verify-installation)
3. [Setup a repo](#setup-a-repo)
   - [

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

## Setup a repo
### Create a repo
Open the folder containing your game's `.uproject` file.

[uproject-image-1]

Click on an empty space in the path at the top. Type in `cmd` and then press <kbd>Enter</kbd>. This will open a terminal window pointing to this folder.

[uproject-image-2]

Type `git init` and then press <kbd>Enter</kbd>. This will initialize a Git repo in this folder.

```console
C:\Games\MyGame\MyGame>git init
Initialized empty Git repository in C:/Games/MyGame/MyGame/.git/
```

At this point, no files are tracked. A file is "tracked" when Git begins monitoring the file for changes.

We'll start adding files to be tracked soon but first, we need to create a `.gitignore` file so that only relevant files are tracked. We'll also need to create a `.gitattributes` file which will tell Git that certain file types should be handled by Git LFS instead of Git.

### Add `.gitignore`
This file tells Git which files should _not_ be tracked. When you clone the repo somewhere else, only tracked files will carry over.

Unreal Engine generates many files that do not need to be tracked. Whenever these "intermediate" files are missing, Unreal Engine will regenerate them. For this reason, we want to make sure only relevant content are tracked and let Unreal Engine regenerate everything else. This significantly reduces the size of the repo.

Open your favorite text editor. Copy and paste everything below and save it as a new file named `.gitignore` in the same folder as your game's `.uproject` file (for example, `MyGame.uproject`).

TODO: embed gist here
