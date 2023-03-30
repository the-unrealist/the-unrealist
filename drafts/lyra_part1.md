# Lyra Deep Dive - Part 1: Introduction

## Introduction
[Lyra](https://docs.unrealengine.com/5.1/en-US/lyra-sample-game-in-unreal-engine/) is a sample game created by Epic Games to demonstrate Unreal Engine frameworks.

This is the first part in a series of technical deep dives for Lyra. In these deep dives, I'll recreate the essence of Lyra step-by-step from scratch focusing on one feature at a time. I have no intention of 
covering everything Lyra has to offer, but I will keep going until when I've felt like I've gone far enough, which may be an arbitrary decision. 😊

For this deep dive series, I will assume the reader (that's you) can read C++ and has more than just a surface level understanding of Unreal Engine.

Articles like this one are essentially my personal notes I've written while exploring Unreal Engine and cleaned up for everybody else to read. Unreal Engine is *massive* and Lyra introduces numerous new features on top of an already massive codebase. It helps me to better understand (and remember) how to do things and how they work if I flesh out my notes to make them useful for not just myself but everyone else too.

I will omit parts of the code when they are not yet relevant. Omitted code may be replaced with a comment so that you'll know there's more to the code than what I've shown. In fact, all comments from me in the code will have the `Citrus:` prefix.

As Lyra is still under development, it's probable for some details in this series to become out of date.

**All code provided in this series are excerpts of Lyra source code that are copyrighted by Epic Games and subject to the 
[Unreal Engine End User License Agreement](https://www.unrealengine.com/en-US/eula/unreal).**

## Chapter 1: The Initial Project
**[Chapter Source Code](https://github.com/the-unrealist/lyra-deep-dive)**

Let's begin with a blank C++ project with four targets:

|Target Name|Outcome|
|-----------|-------|
|`LyraEditor`|Produces an editor build that includes editor-only content. Use this target to launch the Unreal Editor.|
|`LyraServer`|TODO|
|`LyraClient`|TODO|
|`LyraGame`|TODO|

`LyraGameEOS` is another target in Lyra but let's skip that for now. It's a variant of `LyraGame` with specific config overrides for [Epic Online Services](https://dev.epicgames.com/en-US/services).
