+++
title = "Fix Graphical Glitch with Nice DCV"
date = "2023-08-12T19:35:07+10:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["nicedcv"]
keywords = ["nice", "dcv", "nice dcv", "aws"]
description = "Having weird graphical glitches with Nice DCV client on Linux? This could be the fix..."
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

Having weird graphical glitches with Nice DCV client on Linux? This could be the fix...

Here is an example of the weirdness I've been getting using Nice DCV client

![dcv graphical glitch example](/posts/dcv-graphical-glitch-fix/dcv-client-linux-glitch.png)

It seems to happen regardless of which version of the client Im using, I've tested 2023.0, 2022.2, 2021.3... they all do the same thing.

Turns out the issue is with the AMD/Intel GPU (a common combo in laptops) and Nice DCV being launched with the Intel intergrated GPU instead of the AMD discrete GPU.

So the solution is simple, just make the Nice DCV client launch with the discrete GPU.

The easiest way to do that is to modify the shortcut that launches Nice DCV by adding the following line to the end of the file.

```
PrefersNonDefaultGPU=true
```

Here is a simple script to modify the shortcut

```bash
echo PrefersNonDefaultGPU=true >> /usr/share/applications/com.nicesoftware.DcvViewer.desktop
```
