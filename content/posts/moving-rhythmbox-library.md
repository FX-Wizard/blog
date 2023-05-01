+++
title = "Moving Rhythmbox Library"
date = "2023-04-30T23:02:13+10:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["rhythmbox"]
keywords = ["rhythmbox", "music", "library"]
description = "A simple guide to moving your Rhythmbox music library"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

If you've ever found yourself needing to move all your music files, but dont want to have to add them all to Rhythmbox again this is for you. 

Unfortunately at the time of writing Rhythmbox does not have a built-in method to move or repath music within its library. This means that if you want to move your music to a new location or change the file paths, you'll have to do it yourself. Luckily its easy enough and doesn't take too long.

### Move the music files
Moving the files themselves is easy as coping them from their old location to the new one. This can easily be done by copying them to the new location just like any other file. Make sure to note the file path where you are moving them to so you can update Rhythmbox's library later.

### Update the library
To do this you'll need to edit the xml file where Rhythmbox stores the locations of all the files in the library.

First, locate the rhythmdb.xml file. This file is typically located at 
`~/.var/app/org.gnome.Rhythmbox3/data/rhythmbox/rhythmdb.xml`. 

Inside this file is the path to all of the songs. Each song has an `entry` in the XML file. Within that entry is a `location` tag that contains the path to the song file. This is what we want to update with the new path.

```xml
<entry type="song">
  <title>Never Gonna Give You Up</title>
  <genre>Other</genre>
  <artist>Rick Astley</artist>
  <album>Rickroll</album>
  <track-number>6</track-number>
  <duration>214</duration>
  <file-size>3438218</file-size>
  <location>file:///mnt/data/music/Never%20Gonna%20Give%20You%20Up.mp3</location>
  <mtime>1413099759</mtime>
  <first-seen>1652698825</first-seen>
  <last-seen>1682857913</last-seen>
  <bitrate>128</bitrate>
  <date>726103</date>
  <media-type>audio/mpeg</media-type>
  <composer>Unknown</composer>
</entry>
```

To update the file paths, open the file in your favorite text editor such as VS Code, gedit, VIM... then find and replace on all the music file paths. The old file paths should be replaced with the new file paths. Once complete, don't forget to save the file.

**For example** if the old path was

`/mnt/data/music/Never%20Gonna%20Give%20You%20Up.mp3` 

and the new path is

`/home/user/music/Never%20Gonna%20Give%20You%20Up.mp3`

Then find `/mnt/data` and replace with `/home/user`

Its really that simple.