# Lyra Deep Dive - Part 1: Introduction

## Introduction
[Lyra](https://docs.unrealengine.com/5.1/en-US/lyra-sample-game-in-unreal-engine/) is a sample game created by Epic Games to demonstrate Unreal Engine frameworks.

This is the first part in a series of technical deep dives for Lyra. In these deep dives, I'll recreate the essence of Lyra step-by-step from scratch. I have no intention of 
covering everything Lyra has to offer, but I will keep going until when I've felt like I've gone far enough, which may be an arbitrary decision. 😊

Articles like this one are essentially my personal notes I've written while exploring Unreal Engine and cleaned up for everybody else to read. Unreal Engine is *massive* and Lyra introduces numerous new features on top of an already massive codebase. It helps me to better understand (and remember) how to do things and how they work if I flesh out my notes to make them useful for not just myself but everyone else too. This deep dive assumes the reader can read C++ and has more than just a surface level understanding of Unreal Engine.

I will omit parts of the code when they are not yet relevant. Omitted code usually will be replaced with a comment so that you'll know there's more to the code than what I've shown. As Lyra is still under development, it's probable for some sections in this series to become out of date.

**All code provided in this series are excerpts of Lyra source code that are copyrighted by Epic Games and subject to the 
[Unreal Engine End User License Agreement](https://www.unrealengine.com/en-US/eula/unreal).**

## Chapter 1: The Initial Project
**[Chapter Source Code](https://github.com/the-unrealist/lyra-deep-dive)**

Let's begin with a blank C++ project that contains a module for the core game and another module for editor features. 

We'll also need to create five different targets:

|Target Name|Outcome|
|-----------|-------|
|`LyraEditor`|Produces an editor build that includes editor-only content. Use this target to launch the Unreal Editor.|
|`LyraServer`|TODO|
|`LyraClient`|TODO|
|`LyraGame`|TODO|
|`LyraGameEOS`|TODO|
