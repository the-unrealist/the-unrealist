---
layout: post
tags: unreal lyra deep-dive experiences
title: "Lyra Deep Dive - Chapter 2: Experiences"
categories: [Unreal Engine,Lyra]
author: Matt
excerpt: "Lyra introduces the concept of Experiences which are essentially modular game modes. In this chapter, we'll walk through how the Experiences system is implemented."
---

<img src="https://img.shields.io/badge/Unreal%20Engine-5.1-informational" alt="Written for Unreal Engine 5.1"> <img src="https://img.shields.io/badge/-C%2B%2B-orange" alt="C++">

This is the second chapter in the [Lyra Deep Dive](https://unrealist.org/lyra-part-1) series.

## What are Experiences?
TODO: Mention that most of the code is in the `GameModes` folder

## Source Code
[View the source code for this chapter ❭](https://github.com/the-unrealist/lyra-deep-dive/tree/chapter2-experiences)

I recommend looking at the [diff](https://github.com/the-unrealist/lyra-deep-dive/compare/chapter2-experiences...chapter1-introduction) to see what's changed since the previous chapter.

## Plugins
### Game Features
TODO

### Modular Gameplay
TODO

### Modular Gameplay Actors
TODO

## Experience Definitions
### Experience Definition Data Asset
TODO

### Action Sets
TODO

### User-Facing Experience Definition Data Asset
TODO

## Lifecycle Stage 1: Replicated Experience
TODO: explain how an experience is replicated to all players. By the end of this section, the experience definition is replicated to all clients when changed.

### Experience Manager Component
TODO

### Lyra Game State
TODO

### Lyra Asset Manager
TODO

## Stage 2: Load Experience Assets
TODO: Explain how an experience is loaded. By the end of this section, the experience definition asset and asset sets are loaded, and then all game feature plugins referenced by the definition and any asset sets are loaded too.

### Experience Manager
TODO

### Experience Manager Component
TODO

## Stage 3: Execution
TODO: Execute the experience definition's game feature actions and notify events.

### Experience Manager Component
TODO

## Stage 4: Deactivate Experience
TODO

## Pawn Spawning and World Settings
### Lyra Game Mode
TODO

### Lyra World Settings
TODO

## Default Experience
TODO: Create default Lyra experience definition, and set world settings.

## Blueprint Nodes
TODO: Talk about `AsyncAction_ExperienceReady` and any other blueprint nodes related to Lyra Experiences.
