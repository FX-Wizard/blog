+++
title = "Blender Headless Render Node"
date = "2023-08-15T23:24:31+10:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["blender", "linux", "how to"]
keywords = ["blender", "render", "render farm", "cli", "headless", "linux"]
description = "Short guide on how to run Blender without a GUI (graphical user interface) for rendering on Linux."
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

This is a short guide on how to run Blender without a GUI (graphical user interface) for rendering on Linux.

But why would you want run Blender without a GUI? Simple, if your setting up a dedicated render node you don't want to waste precious RAM or GPU power on running a graphical desktop environment that never gets used.

This guide assumes you already have Linux installed without a GUI. If not, go grab a copy of [Ubuntu Server](https://ubuntu.com/download/server) and come back once its installed.


## Download

*How to download an application onto a computer that has no desktop or even a web browser?*

Use another computer that has a web browser and go to the [Blender downloads](https://www.blender.org/download/) page, then find the download button.

Instead of left clicking on the download button to download, `right click`, then click on `Copy Link`. This will copy the download URL to the clipboard.

Using ssh remote into the Linux render node, then use the curl command and paste the link you copied to download Blender to the render node.

For example.
```bash
curl -O https://www.blender.org/download/release/Blender3.6/blender-3.6.1-linux-x64.tar.xz/
```

Alternatively you could download Blender and use scp to copy it over. Or just use an old fashion USB drive.

## Installation

Blender on Linux doesn't have an installer, is just a zipped file with Blender in it all ready to go.

Extract the contents of the file.
```bash
tar xvf blender-*.tar.xz
```

And that's it! Blender is now installed. But don't go just yet, there are a few extra things Blender needs to run.

If you try to run a Blender render now you might get an error like this one.
```
error while loading shared libraries: 
```

Because there is no graphical environment and Blender is an application that produces graphics it needs a few extra libraries to be installed before it can render.

Here is the list of packages Blender needs installed.

Debian or Ubuntu
```bash
sudo apt install -y libxrender1 libxxf86vm1 libxfixes3 libxi6 libxkbcommon0 libsm6 libgl1 libegl1
```

Rocky Linux or Fedora
```bash
sudo dnf install -y libXxf86vm libXfixes libXi libSM libGL
```

And that's really it this time.

Happy rendering!