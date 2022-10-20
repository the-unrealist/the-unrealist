---
layout: post
tags: unreal commonui umg
title: "Common UI: Action Bar"
categories: [Common UI]
author: Matt
---

When the user interface is open in most modern games, you'll find a list of actions available to the player.
This is typically a horizontal bar located at the bottom of the screen.

Here's a few:

**Destiny**

<img src="/assets/images/action-bar-destiny2.png" alt="Screenshot of the action bar in Destiny 2 showing the following actions: Toggle Subscreen bound to the S key, Gear Stats bound to the E key, and Dismiss bound to the Escape key">

**Animal Crossing: New Horizons**

<img src="/assets/images/action-bar-animal-crossing.png" alt="Screenshot of the action bar in Animal Crossing showing the following actions: Take Off bound to the X button, Cancel bound to the B button, Change bound to the A button, and Done bound to the plus button">

And, of course... **Fortnite**! 

<img src="/assets/images/action-bar-fortnite.png" alt="Screenshot of the action bar in Fortnite showing the following actions: View Match Stats bound to the V key, Report Player not bound to any key, and Leave Match not bound to any key">

The Common UI plugin was originally developed for Fortnite and the action bar we'll be creating in this tutorial is the same as the one used in the game! 
In case you're not aware, this article is part of a series about the [Common UI plugin](https://docs.unrealengine.com/5.0/en-US/common-ui-plugin-for-advanced-user-interfaces-in-unreal-engine/).
Common UI is a cross-platform UI plugin developed by Epic Games for Unreal Engine.

## Set-up
### 1. Common Bound Action Button
First, we'll need to build our own button widget that will be used by the action bar for each available action. Go ahead and create a new Widget Blueprint based on `CommonBoundActionButton`.

<img src="/assets/images/new-widget-commonboundactionbutton.png" alt="A screenshot showing that CommonBoundActionButton is selected as the widget blueprint's parent class">

Just like with the `CommonButtonBase`, we'll need to provide a layout for the button. In the **Bind Widgets** panel, the properties that are required by the parent `CommonBoundActionButton` class are listed here.

<img src="/assets/images/bound-action-button-bind-widgets.png" alt="Bind Widgets panel showing a Common Text property named Text_ActionName is required, and a Common Action Widget property named InputActionWidget is optional.">

This means our button must have a Common Text widget and it must be named `Text_ActionName`, or else, it will fail to compile. 

Let's start off with an Overlay and a Common Text.

<img src="/assets/images/bound-action-button-initial-layout.png" alt="A screenshot of the hierarchy panel for the button showing a Common Text widget wrapped in an Overlay panel.">

At this point, if we go back to the Bind Widgets panel, we will see a checkmark indicating that the property is bound.

<img src="/assets/images/bound-action-button-bind-widgets-2.png" alt="Bind Widgets panel showing a checkmark next to the Common Text property.">

This part is optional, but in this tutorial we'll add a Common Action Widget, name it `InputActionWidget`, and put it in a Horizontal Box along with the text.

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
  <tr><td>Icon Rim Brush</td><td>The image or material that's drawn behind the button icon. Typically used to display the button outline.</td></tr>
 </tbody>
</table>

### 2. Common Bound Action Bar
Put the Common Bound Action Bar wherever it makes sense in a widget blueprint. A good place is in your game's root UI container widget. For this tutorial, we'll put it underneath a Common Activatable Widget Stack that we'll use to push activatable widgets.

<img src="/assets/images/action-bar-sample-layout.png" alt="A screenshot of the widget blueprint designer showing a Vertical Box with a Common Activatable Widget Stack and a Common Bound Action Bar.">

Under **Entry Layout** in the Details panel, set the **Action Button Class** to the button widget we created earlier. Under **Dynamic Entry Box**, the **Entry Box Type** property sets how the buttons are laid out. This can be further customized with the properties under the Entry Layout category.

The action bar works out of the box so there's nothing else you need to do here.

## Registering Actions
TODO

## Button Icons
TODO

## Progress Indicator Material Example
TODO
