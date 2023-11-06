+++
title = "Blender Deadline Submitter Installation"
date = "2023-08-09T20:08:44+10:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["DEADLINE", "BLENDER", "RENDER FARM"]
keywords = ["thinkbox", "deadline", "blender", "render farm"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

This guide focuses on Blender 3.6.1, however any version from 2.80 onwards should work.

## Getting the Deadline Add-on

There are 2 ways to get the add-on

**1. Download from Monitor**

Open the Deadline Monitor and in the top menu bar click

Tools -> Download Integrated Submission Scripts...

![deadline download submitter script 1](/posts/blender-deadline/deadline-get-submitter-1.png)

Select Blender check box, then in the 'Download To' field enter the path to where you want to download the submitters. Then Click 'OK'

![deadline download submitter script 2](/posts/blender-deadline/deadline-get-submitter-2.png)

This will create a folder in the select destination called `submission` that contains the submitter scripts

**2. Copy from Repository**

If you are using a direct connect or have shared the repository on a network drive its easy to copy the submitter files directly out of the repository itself.

In the repository there is a folder called `submission` that contains all the submitter scripts.


Its also possible to copy the submitters from the repository using other methods such as scp.

For the Linux install of the Deadline Repository the default location for the submitter scripts is
`/opt/Thinkbox/DeadlineRepository10/submission`

---

## Installing Deadline Add-on

There are a few different ways to install the submitter script depending on how much automation you need and your pipeline.

### Manual add-on installation

This is the typical installation method and best suited to individual users.

**Installation**

Use add-ons menu to install the python script

Go to Edit -> Preferences

![blender preferences menu](/posts/blender-deadline/blender-deadline-preferences-menu.png)

In the preferences window click 'Add-ons' then click 'Install...'

![blender preferences add-ons window](/posts/blender-deadline/blender-preferences-addons.png)

In the File View window navigate to where you downloaded the submitter script. Within the downloaded submitter open submission/Blender/Client/DeadlineBlenderClient.py then click 'Install Add-on'

![blender deadline add-on](/posts/blender-deadline/blender-deadline-addon.png)

With the add-on installed, search for 'deadline' and tick 'Submit Blender To Deadline' to enable the add-on in Blender

![blender enable deadline add-on](/posts/blender-deadline/blender-deadline-enable-addon.png)


### Copy script to script directory

The advantage of this method is it can be set anywhere, including on a shared drive so other instances of Blender can all read the same scripts. This is quite useful for a studio with many artists all needing to work on the same projects with the same scripts.

set script path in preferences

![blender script path setting](/posts/blender-deadline/blender-deadline-script-paths.png)

In this example the blender scripts are being stored in a folder called scripts.

Create a folder called `addons` and copy `DeadlineBlenderClient.py` into it.

The structure of the scripts folder should look like this.
```
scripts /
|-- addons /
|  `-- DeadlineBlenderClient.py
```

If done correctly Blender will be able to find the add-on and you can enable it in the add-ons preferences window.

![blender enable deadline add-on](/posts/blender-deadline/blender-deadline-enable-addon.png)


### Installer add-on installation

Deadline also provides a handy installer. Assuming the application is using the default script paths this is a convenient way to install the add-on. If you are CNA certified you'll need no instruction to use this method.

> Note: CNA (click next always)

---

## Submit render to Deadline

Once the submitter is installed you can submit a scene 

![blender submit render to deadline menu](/posts/blender-deadline/blender-deadline-submit-render-menu.png)

![deadline submitter window](/posts/blender-deadline/blender-deadline-submitter-window.png)