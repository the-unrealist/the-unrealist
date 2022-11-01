---
layout: post
tags: unreal commonui umg
title: "Common UI: Action Bar"
categories: [Common UI]
author: Matt
excerpt: "When the user interface is open in most modern games, you'll find a list of actions available to the player. This is typically a horizontal bar located at the bottom of the screen. This tutorial explains how to create one with the Common UI plugin."
---

When the user interface is open in most modern games, you'll find a list of actions available to the player. This is typically a horizontal bar located at the bottom of the screen.

Here are some examples:

**Destiny 2**

<img src="/assets/images/action-bar-destiny2.png" alt="Screenshot of the action bar in Destiny 2 showing the following actions: Toggle Subscreen bound to the S key, Gear Stats bound to the E key, and Dismiss bound to the Escape key">

**Animal Crossing: New Horizons**

<img src="/assets/images/action-bar-animal-crossing.png" alt="Screenshot of the action bar in Animal Crossing showing the following actions: Take Off bound to the X button, Cancel bound to the B button, Change bound to the A button, and Done bound to the plus button">

And, of course... **Fortnite**! 

<img src="/assets/images/action-bar-fortnite.png" alt="Screenshot of the action bar in Fortnite showing the following actions: View Match Stats bound to the V key, Report Player not bound to any key, and Leave Match not bound to any key">

The Common UI plugin was originally developed for Fortnite and the action bar we'll be creating in this tutorial is the same as the one used in the game! 
In case you're not aware, this article is part of a series about the [Common UI plugin](https://docs.unrealengine.com/5.0/en-US/common-ui-plugin-for-advanced-user-interfaces-in-unreal-engine/).
Common UI is a cross-platform UI plugin developed by Epic Games for Unreal Engine.

## Table of Contents
* [Setup](#setup)
  * [1. Common Bound Action Button](#1-common-bound-action-button)
  * [2. Common Bound Action Bar](#2-common-bound-action-bar)
* [Input Action Icons](#input-action-icons)
* [Registering Actions](#registering-actions)

## Setup
### 1. Common Bound Action Button
First, we'll need to build our own button widget that will be used by the action bar for each available action. Go ahead and create a new Widget Blueprint based on `CommonBoundActionButton`.

<img src="/assets/images/new-widget-commonboundactionbutton.png" alt="A screenshot showing that CommonBoundActionButton is selected as the widget blueprint's parent class">

Just like with the `CommonButtonBase`, we'll need to provide a layout for the button. In the **Bind Widgets** panel, the properties required by the parent `CommonBoundActionButton` class are listed here.

<img src="/assets/images/bound-action-button-bind-widgets.png" alt="Bind Widgets panel showing a Common Text property named Text_ActionName is required, and a Common Action Widget property named InputActionWidget is optional.">

This means our button must have a Common Text widget and it must be named `Text_ActionName`, or else, it will fail to compile. 

Let's start off with an Overlay and a Common Text.

<img src="/assets/images/bound-action-button-initial-layout.png" alt="A screenshot of the hierarchy panel for the button showing a Common Text widget wrapped in an Overlay panel.">

At this point, if we go back to the Bind Widgets panel, we will see a checkmark indicating that the property is bound.

<img src="/assets/images/bound-action-button-bind-widgets-2.png" alt="Bind Widgets panel showing a checkmark next to the Common Text property.">

This part is optional, but in this tutorial we'll add a Common Action Widget, name it `InputActionWidget`, and put it in a Horizontal Box along with the text. This widget will automatically display the input icon associated with the action.

<img src="/assets/images/bound-action-button-layout-with-icon.png" alt="A screenshot of the hierarchy panel for the button showing a Common Action Widget and a Common Text widget wrapped together in a Horizontal Box.">

The Bind Widgets panel will update to confirm that the icon is also bound.

<img src="/assets/images/bound-action-button-bind-widgets-4.png" alt="Bind Widgets panel showing a checkmark next to the Common Text property and the Common Action Widget property.">

There are additional properties that can be set in the Details panel for the Common Action Widget.

<img src="/assets/images/common-action-widget-details.png" alt="Details panel showing properties under the Common Action Widget category. The properties are Progress Material Brush, Progress Material Param, Icon Rim Brush, and an array of Input Actions.">

<table>
 <thead>
  <tr><th>Property</th><th>Description</th></tr>
 </thead>
 <tbody>
  <tr><td>Progress Material Brush</td><td>The material used to draw the progress indicator for actions that require holding down input. This is drawn on top of the button icon.</td></tr>
  <tr><td>Progress Material Param</td><td>The name of a scalar parameter used by the material. A percentage value between 0-1 will be provided via this parameter.</td></tr>
  <tr><td>Icon Rim Brush</td><td>The image or material that's drawn behind the button icon.</td></tr>
 </tbody>
</table>

### 2. Common Bound Action Bar
Put the Common Bound Action Bar wherever it makes sense in a widget blueprint. A good place is in your game's root UI container widget. In this tutorial, we'll put it underneath a Common Activatable Widget Stack that we'll use to push activatable widgets.

<img src="/assets/images/action-bar-sample-layout.png" alt="A screenshot of the widget blueprint designer showing a Vertical Box with a Common Activatable Widget Stack and a Common Bound Action Bar.">

Under **Entry Layout** in the Details panel, set the **Action Button Class** to the button widget we created earlier. Under **Dynamic Entry Box**, the **Entry Box Type** property sets how the buttons are laid out. This can be further customized with the properties under the Entry Layout category.

The action bar works out of the box so there's nothing else you need to do here.

## Input Action Icons
Let's do a quick run through on how to set up input action icons for each controller type.

Create a Blueprint and select `CommonInputBaseControllerData` as the parent class. 

<img src="/assets/images/common-input-base-controller-data.png" alt="New Blueprint dialog with the Common Input Base Controller Data selected as the parent class">

We'll create one to represent the Generic gamepad which is used as the default in case we do not have one specific to the gamepad that's being used. Import an icon for each button as a texture.

This needs to be done for each input device you want to support.

<img src="/assets/images/generic-gamepad-brushes.png" alt="Content browser for a folder containing a Common Input Base Controller Data asset named Generic Gamepad Brushes and a texture representing each one of the four face buttons of a gamepad.">

In the Blueprint, add an entry for each input type to the **Input Brush Data Map**.

<img src="/assets/images/generic-gamepad-brushes-data.png" alt="Details view for the Common Input Base Controller Data asset with the Input Brush Data Map expanded to show entries mapping each one of the four face buttons of a gamepad to its texture.">

For the Generic gamepad, the **Input Type** is set to **Gamepad** and the **Gamepad Name** is set to **Generic**. If you have source code access to consoles in Unreal Engine, then you'll see other options for gamepads belonging to specific consoles such as the Xbox or Nintendo Switch here.

<img src="/assets/images/generic-gamepad-brushes-data-2.png" alt="Details view for the Common Input Base Controller Data asset with the Input Type set to Gamepad and the Gamepad Name set to Generic.">

Now, open **Project Settings > Game > Common Input Settings**. For each supported platform (Windows in this example), add a reference to the relevant Controller Data assets. On platforms that support generic gamepads (e.g., Windows), make sure the **Default Gamepad Name** is set to **Generic**, otherwise, use the selected name in the Controller Data asset.

<img src="/assets/images/project-settings-common-input-settings.png" alt="Common Input Settings in Project Settings showing the Controller Data array with one entry pointing to Generic Gamepad Brushes asset and another pointing to Keyboard Brushes asset.">

At this point, Common Acton Widgets will automatically display the appropriate icon based on the active input device.

## Registering Actions
### Back Handler
**Common Activatable Widgets** can register a binding for the default Back input action by setting **Is Back Handler** to true. The default behavior is to deactivate the widget. This can be changed by overriding the **On Handle Back Action** function of the Common Activatable Widget. 

To display the Back input action in the Action Bar, both **Is Back Action Displayed in Action Bar** *and* **Display in Action Bar** need to be set to true.

### Custom Actions
While it's straightforward to handle the Back input, it gets a rather tricky if you want to bind more inputs. This will require C++, but there is a workaround if you want to do it in Blueprints only. The workaround is to add invisible (zero-width) Common Buttons to a Common Activatable Widget. 

Otherwise, you'll need to jump into C++ and call `RegisterUIActionBinding` in `UCommonUserWidget`, which is, unfortunately, not exposed to Blueprints.

It's not difficult to resolve this issue. My solution is to create a C++ class based on `UCommonActivatableWidget` and add `BlueprintCallable` functions so that I can register input bindings entirely in Blueprints as you can see below:

<img src="/assets/images/extendedactivatablewidgetfunctions3.png" alt="Screenshot of custom Blueprint functions: Register Binding, Unregister Binding, and Unregister All Bindings">

First, you'll need to add `CommonUI` and `CommonInput` to `PublicDependencyModuleNames` for your game's module. Then, add `ExtendedCommonActivatableWidget.h` and `ExtendedCommonActivatableWidget.cpp` to your game's source.

{% gist 0297fbda19afe1e4d0f3afba92104ffd %}

The purpose of `FInputActionBindingHandle` is to represent an opaque Blueprintable handle provided by `RegisterBinding` that can be used to unregister the binding. This is optional as all bindings created by the widget will be unregistered when it's destroyed, but can be useful in some cases.

Hope this helps!

<img src="/assets/images/action-bar-demo.png" alt="Screenshot of an Action Bar created with my custom C++ class that I shared above">
