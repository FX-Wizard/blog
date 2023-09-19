+++
title = "Batch Convert Audio Files"
date = "2023-07-01T22:29:29+10:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["audacity", "audio"]
keywords = ["audacity", "audio", "batch", "convert"]
description = "Batch convert audio files to a different format using Audacity"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

Need to convert a whole bunch of audio files to a different format? Here's how!

We are going to use a free open source tool called [Audacity](https://www.audacityteam.org/) to perform the conversion. Audacity has a handy macros feature that 

Downloading and installing Audacity is easy, and I'm sure you are capable of installing apps so I wont waste your time explaining it.

---

### Using the Macro Manager

In the top menu bar go to `Tools` -> `Macro Manager` which will open the Manage Macros window.

On the left hand side is a list of macros and on the right is the steps that macro will take when processing the file.

Select `MP3 Conversion` then down the bottom of the window, click `Files...` and open the folder with the audio files you wish to convert.

![audacity macro output preferences](/posts/batch-convert-audio-files/audacity-macro-manager.png)

---

### Changing macro output folder

When running a macro it does not ask where you want to save the outputted files and instead leaves them in a default location.

Here is how you can change the default output folder.

In the top bar click `Edit` -> `Preferences` (keyboard shortcut ctrl + p)

Then in the side left section click `Directories`. There will be a field called `Macro output:` where you can set the folder where the macros will be saved.

![audacity macro output preferences](/posts/batch-convert-audio-files/audacity-macro-output-preferences.png)

---

### Creating your own custom macro

The MP3 Conversion macro provided with Audacity runs a normalisation on the audio before exporting it. This can alter how the output sounds, which might not be desirable. To output the audio files without normalising them we can create a custom macro that skips that step.

Click `New` to create a new macro

![audacity macro output preferences](/posts/batch-convert-audio-files/audacity-custom-macro-1.png)

Give the macro a name, for example "MP3 Direct Conversion"

![audacity macro output preferences](/posts/batch-convert-audio-files/audacity-custom-macro-2.png)

Click `Insert` to add a new step to the macro

![audacity macro output preferences](/posts/batch-convert-audio-files/audacity-custom-macro-3.png)

Select `Export as MP3` command then `OK`

![audacity macro output preferences](/posts/batch-convert-audio-files/audacity-custom-macro-4.png)

Once you have added al lthe steps you want for your macro, click `Save`

![audacity macro output preferences](/posts/batch-convert-audio-files/audacity-custom-macro-5.png)

---

Now you can batch convert audio files