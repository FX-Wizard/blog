+++
title = "Using A Drawing Tablet With GIMP"
date = "2024-02-04T11:21:40+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["GIMP", "WACOM", "DRAWING TABLET", "HOW TO"]
keywords = ["gimp", "wacom", "drawing", "art", "drawing tablet", "how to"]
description = "Quick set up guide to get GIMP working with a drawing tablet"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

Simple how to guide to get your drawing tablet's pen pressure and tilt working with GIMP.

I've used a Wacom Intuos Pro when making this guide but any [supported ped device](https://developer.gimp.org/core/specifications/graphic_tablets_support/) should work fine.

## How To

### Enable Pen

In the top menu bar go to `Edit` -> `Input Devices`

On the left side all the input devices detected by GIMP are listed. 

1. Select the Pen input device you want to enable.
2. Change the mode to 'Screen'.
3. Click Save to apply the settings

Repeat this process for any other input devices you want to enable such as the pen eraser.

![GIMP Configure Input Devices](/posts/drawing-tablet-gimp-setup/gimp-configure-input-devices.png)

### Turn On Pen Dynamics

1. On the left hand menu, select the 'Tool Options' tab.
2. Change the setting to 'Basic Dynamics'.


![GIMP Enable Pen Dynamics](/posts/drawing-tablet-gimp-setup/gimp-tablet-turn-on-dynamics-basic.png)


For more advanced options click the button highlighted bellow to open the 'Pain Dynamics Editor'.

![GIMP Pen Dynamics Advanced Options Button](/posts/drawing-tablet-gimp-setup/gimp-pen-dynamics-advanced-settings-button.png)

The editor usually opens on the right side.

You can change what dynamic effects are applied by the pen in this menu.

![GIMP Pen Dynamics Advanced Options](/posts/drawing-tablet-gimp-setup/gimp-pen-advanced-dynamics-settings.png)

---

Happy Drawing!