---
layout: post
tags: unreal commonui umg
title: "Common UI: Switchers and Tabs"
categories: [Common UI]
author: Matt
---

In this article, we'll learn about the types of switchers in the [Common UI plugin](https://docs.unrealengine.com/5.0/en-US/common-ui-plugin-for-advanced-user-interfaces-in-unreal-engine/) and how to leverage them to create tabs.

Common UI is a cross-platform UI plugin developed by Epic Games for Unreal Engine 5. There are few [talks](https://www.youtube.com/watch?v=TTB5y-03SnE) on YouTube covering the basics of Common UI and [how it's used in Lyra](https://www.youtube.com/watch?v=u06GAVxyIag), but they haven't explained how to create **switchers**, **tabs**, and **carousels**. There is a lack of documentation for Common UI that I'm hoping to help fill in here.

For that reason, **this tutorial assumes you have a basic understanding of Common UI including setting up inputs and styling**.

## Table of Contents
* [Switchers](#switchers)
  * Common Animated Switcher
  * Common Activatable Widget Switcher
  * Common Visibility Switcher
* [Tabs](#tabs)
  * Creating a Common Tab List Widget
  * Linking a Tab List Widget to a Switcher
  * Adding Gamepad Input Actions
* [Carousels](#carousels)
  * Common Widget Carousel

## Switchers
A switcher is a widget that displays one child widget at a time and can switch to another child widget. It can act as a standalone widget or be linked to a tab list for a tabbed experience.

<details open style="margin-bottom: 1em;">
  <summary class="toggle-link">Show/hide animated preview</summary>
  <img src="/assets/images/basic-switcher.gif" style="height: 384px;" alt="Animated GIF demonstrating a switcher using the default Fade transition effect as it switches between 4 visually distinct panels">
  <p><small><a href="https://www.flaticon.com/free-icons/fruit" title="fruit icons">Fruit icons created by Prosymbols - Flaticon</a></small></p>
</details>

The **[Common Animated Switcher](https://docs.unrealengine.com/5.0/en-US/API/Plugins/CommonUI/UCommonAnimatedSwitcher/)** is derived from UMG Widget Switcher and animates the transition between child widgets.

At the time of writing, this widget describes itself as "a widget switcher that activates / deactivates CommonActivatableWidgets, allowing for associated animations to trigger."

This is incorrect! The Common Animated Switcher does *not* activate widgets.

That's where the **[Common Activatable Widget Switcher](https://docs.unrealengine.com/5.0/en-US/API/Plugins/CommonUI/UCommonActivatableWidgetSwitcher/)** comes in. It is derived from the Common Animated Switcher, and makes it so that it _does_ activate widgets.

Keep in mind that the next widget is not activated until the previous widget has transitioned fully out of view.
<table>
 <thead>
  <tr><th>Activatable Widget Event</th><th>Trigger</th></tr>
 </thead>
 <tbody>
  <tr><td>On Activated</td><td>Previous widget is fully out of view. This widget will be active while it's appearing.</td></tr>
  <tr><td>On Deactivated</td><td>Immediately when switching. This widget will be inactive while it's disappearing.</td></tr>
 </tbody>
</table>

For both widgets above, there are four transition animations you can use:
<table>
 <thead>
  <tr><th>Transition</th><th>Behavior</th></tr>
 </thead>
 <tbody>
  <tr><td>Fade Only</td><td>Fade transition only with no movement</td></tr>
  <tr><td>Horizontal</td><td>Increasing the active index goes right, decreasing goes left</td></tr>
  <tr><td>Vertical</td><td>Increasing the active index goes up, decreasing goes down</td></tr>
  <tr><td>Zoom</td><td>Increasing the active index zooms in, decreasing zooms out</td></tr>
 </tbody>
</table>

The only other transition parameters you can adjust are the curve type and duration. Anything else will require modifying `SCommonAnimatedSwitcher.cpp` in the Common UI plugin's C++ source.

For your convenience, here's an overview of all relevant functions you can use in Blueprints.
<img src="/assets/images/switcher-blueprint-functions.png" alt="Screenshot of a collection of Blueprint nodes in clockwise order starting from top left: Activate Next Widget, Activate Previous Widget, Is Currently Switching, Has Widgets, Set Active Widget Index, Set Active Widget, and Set Disable Transition Animation.">

Finally, there's the **[Common Visibility Switcher](https://docs.unrealengine.com/5.0/en-US/API/Plugins/CommonUI/UCommonVisibilitySwitcher/)** which is a bit different from above in that it is derived from UMG Overlay widget and is identical to UMG Widget Switcher in that there are no animations and only one widget can be visible at a time. However, it does activate widgets whenever they become visible.

## Tabs
Coming soon

## Carousels
A carousel is different from a switcher in that both the previous and next widgets are simultaneously visible during the transition. A switcher waits for the previous widget to fade out or move out of sight before bringing in the next widget. A carousel is much more like a scroll box where widgets are scrolled into and out of view.

Common UI has a **[Common Widget Carousel](https://docs.unrealengine.com/5.0/en-US/API/Plugins/CommonUI/UCommonWidgetCarousel/)** that may be optionally linked to a **[Common Widget Carousel Nav Bar](https://docs.unrealengine.com/5.0/en-US/API/Plugins/CommonUI/UCommonWidgetCarouselNavBar/)**.

Unfortunately, I was not able to get it to appear in my UI. It does scroll through widgets as expected so it does technically "work", but for some reason it never gets painted onto the screen in the editor or game. I spent a long time debugging and analyzing the source and still couldn't understand why it's not visible. 

At this point, I believe it's bugged and probably cannot be used yet. If anyone knows how to make it work, please let me know on Twitter: [@unrealist_matt](https://twitter.com/unrealist_matt) :)
