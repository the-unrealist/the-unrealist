---
permalink: /plugins/parallax-panel
title: "Parallax Panel"
---

![Icon for the plugin showing a stylized scene of a mountain with clouds and trees.](https://github.com/the-unrealist/parallax-panel/blob/main/Resources/Icon128.png?raw=true)

## Parallax Panel
A 2D panel widget that simulates a 3D depth effect in user interfaces.

<details open style="margin-bottom: 1em;">
  <summary class="toggle-link" style="cursor: pointer;">Show/hide animated preview</summary>
  <img src="https://github.com/the-unrealist/parallax-panel/blob/main/.images/parallax-preview.gif?raw=true" alt="Animated GIF showing a simple cityscape with each layer moving at different speeds.">
</details>

### Download
[Get the source code from GitHub ❭](https://github.com/the-unrealist/parallax-panel)

### Features
* Simple and lightweight panel
* Any widget can be a layer
* Supports [widget animations](https://docs.unrealengine.com/5.3/en-US/animating-umg-widgets-in-unreal-engine/)

### User Guide
In a widget blueprint, create a Parallax Panel widget from the palette.

![Screenshot showing Parallax Panel categorized under Panel in the Palette window.](https://github.com/the-unrealist/parallax-panel/blob/main/.images/01-palette.png?raw=true)
<br /><br />

Add child widgets to the panel. Each child widget is a layer.

![Screenshot showing a Parallax Panel widget with three different image child widgets in the hierarchy.](https://github.com/the-unrealist/parallax-panel/blob/main/.images/02b-hierarchy.png?raw=true)
<br /><br />

Set the **Distance** of each layer under *Slot (Parallax Panel Slot)* in the Details panel.

![Screenshot showing the Distance property in the details panel for a child widget. The property is marked with a red arrow.](https://github.com/the-unrealist/parallax-panel/blob/main/.images/02-layer-distance.png?raw=true)
<br /><br />

Use the **Offset** property in the Parallax Panel to control the perspective.

![Screenshot showing the Offset property in the details panel for a Parallax Panel widget.](https://github.com/the-unrealist/parallax-panel/blob/main/.images/03-panel-offset.png?raw=true)
<br /><br />

The parallax scrolling equation used in this plugin is $$\text{Transform} = \frac{\text{Offset}}{1+(\frac{\text{Distance}}{100})}$$. A layer with a distance of 100 will move twice less than a layer with a distance of zero.

---------------------

Forest icons created by Freepik - [Flaticon](https://www.flaticon.com/free-icons/forest)
