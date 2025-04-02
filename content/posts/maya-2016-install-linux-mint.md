+++
title = "Maya 2016 Install On Linux Mint"
date = "2016-03-03T13:08:32+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["MAYA", "LINUX", "HOW TO", "ARCHIVE"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++


Installing Maya 2016 on Linux mint is a similar process to installing the 2015 version. However there are a few different steps to get it running.

## Step 1:

We need to install a whole bunch of programs first that Maya needs to be able to run.

```bash
sudo apt-get install -y alien
sudo apt-get install -y csh tcsh libaudiofile-dev libglw1-mesa elfutils
sudo apt-get install -y gamin libglw1-mesa-dev mesa-utils libcurl3
sudo apt-get install -y ttf-liberation xfonts-100dpi xfonts-75dpi
sudo apt-get install -y ttf-mscorefonts-installer
```

Install the following if you plan to run a flexlm licencing server on your computer:

```
sudo apt-get install lsb-core
```

## Step 2:

Next we need to convert the Maya's installer .rpm files into .deb files.

```
sudo alien -cv *.rpm
```

> Note: last time I did this I had a problem with the Composite package installing and making a huge mess, its not necessary for Maya to run so its best to delete it

## Step 3:

Install the newly converted .deb files

dpkg -i *.deb

## Step 4:

Maya is going to complain about not having the correct version of libssl even though it doesn't come with that version, but that can be easily fixed.

```bash
sudo apt-get install libssl1.0.0 libssl-dev
sudo ln -s /lib/x86_64-linux-gnu/libssl.so.1.0.0 /lib/x86_64-linux-gnu/libssl.so.10
sudo ln -s /lib/x86_64-linux-gnu/libcrypto.so.1.0.0 /lib/x86_64-linux-gnu/libcrypto.so.10
```

## Step 5:

Now that its installed run the installer to license the application. It will fail to install but it will add your license to Maya.