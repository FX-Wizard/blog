+++
title = "Maya 2015 Install On Linux Mint"
date = "2015-08-02T09:52:34+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["MAYA", "LINUX", "HOW TO", "ARCHIVE"]
keywords = ["maya", "linux"]
description = "Guide to installing Maya 2015 on Linux Mint"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

Officially Maya on Linux only supports Red Hat, Fedora and CentOS. But that won't stop us from breaking the rules and installing it on unsupported platforms like Linux Mint!

The installation files are in the .rpm (Red Hat Package Manager) format which as its name suggests is for Red Hat based distributions and won't work on Debian based distributions like Linux Mint or Ubuntu. So lets convert them into a .deb package that will work on Linux Mint using an awesome tool called Alien!


### Prerequisits

This guide assumes you already have Linux Mint installed with a copy of Maya 2015 downloaded.


## Installing dependencies:

First install `alien` to convert the .rpm to a .deb.

```bash
sudo apt-get update
sudo apt-get install -y alien
```

Maya needs a few extra packages to work correctly, so lets install them.

```bash
sudo apt-get install -y csh tcsh libaudiofile-dev libglw1-mesa elfutils
sudo apt-get install -y gamin libglw1-mesa-dev mesa-utils xfs xfstt
sudo apt-get install -y ttf-liberation xfonts-100dpi xfonts-75dpi
sudo apt-get install -y ttf-mscorefonts-installer
```

## Convert Install Files

Extract the Maya installer tar ball

```bash
tar zxvf *Maya*2015*.tgz
```

Once extracted, open up the extracted folder and run the `alien` command to convert all the install files into .deb packages.

```bash
sudo alien -k *.rpm
```

## Install Maya

To install simply run `dpkg` to install like a normal Debian package. You'll need to run all the converted install packages to ensure all the license components Maya needs are also installed, or it wont work because it has no license.

```bash
sudo dpkg -i *.deb
```

## License Maya

Now that its installed run the `setup` installer to licence the application.

```bash
sudo ./setup
```

This will take you though the installation wizard that asks for your license keys. It will fail to install but that doesn't matter since we already installed it, all we want the setup app to do is add the license info.


## Bugs

Now that the installation is complete its time to fix all the bugs.

**Bug 1:** Copy across the libraries that were missed in the installation.

```
sudo cp <maya installation directory>/libadlmPIT* /usr/lib/
sudo cp <maya installation directory>/libadlmutil* /usr/lib/
```

**Bug 2:** Add a temp folder for mental ray to cache stuff to

```
sudo mkdir /usr/tmp
sudo chmod 777 /usr/tmp
```

**Bug 3:** Maya is going to complain about not having the correct version of libssl even though it doesn't come with that version, but that can be easily fixed.

```
sudo ln -s /usr/autodesk/maya2015-x64/support/openssl/libssl.so.6 /usr/autodesk/maya2015-x64/lib/libssl.so.10
sudo ln -s /usr/autodesk/maya2015-x64/support/openssl/libcrypto.so.6 /usr/autodesk/maya2015-x64/lib/libcrypto.so.10
```

**Bug 4:** Maya cannot find the systems tiff or jpg libraries because Red Hat puts them in a different place to Debian.

```
sudo apt-get install libjpeg62:i386
sudo ln -s /usr/lib/x86\_64-linux-gnu/libjpeg.so.8.0.2 /usr/lib/libjpeg.so.62
sudo ln -s /usr/lib/x86\_64-linux-gnu/libtiff.so.5 /usr/lib/libtiff.so.3
```

creating a sym link to the files works well in 2014 but I find that it doesn't always work for 2015. So instead we need to modify the maya file in Maya's bin directory.

`sudo vim /usr/autodesk/maya2015-x64/bin/maya`

search for the line:

`setenv LIBQUICKTIME_PLUGIN_DIR "$MAYA_LOCATION/lib"`

Directly under that line add the following:

```
# I added the next line to fix a jpg problem
setenv LD\_PRELOAD /usr/lib/x86\_64-linux-gnu/libjpeg.so.62
```

**Bug 5:** When trying to generate 3d text in maya it wont work because all the fonts on the debian system are the wrong type. To convert them we need to use t1ascii to convert them to the correct type.

```shell
for x in `find /usr/share/fonts -name *.pfb`
do
y=$(sed s/\.[^\.]*$// <<< $x)
t1ascii $x $y.pfa
done
xset +fp /usr/share/fonts/X11/100dpi/
xset +fp /usr/share/fonts/X11/75dpi/
xset fp rehash
```

## Run Maya

Now that Maya 2015 is installed time to run it and make something awesome!