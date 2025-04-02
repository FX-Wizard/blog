+++
title = "Maya 2017 Install On Linux Mint"
date = "2017-05-08T13:08:37+11:00"
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

## Step 1:

Use Linux Mint's built in extraction tools to extract the contents of the .rpm file

## Step 2:

Copy the extracted folders to the following locations:

For Maya 2017:

```
sudo cp -rv opt/Autodesk /opt/
sudo cp -rv usr/autodesk /usr/
sudo cp -rv var/opt/Autodesk /var/opt/
```

For adlmapps:

```
sudo cp -rv opt/Autodesk/Adlm /opt/Autodesk/
sudo cp -rv var/opt/Autodesk/Adlm/* /var/opt/Autodesk/Adlm/
```

For adlmflexnetclient:

```
sudo cp -rv opt/Autodesk/Adlm/FLEXnet /opt/Autodesk/Adlm/
```

For adlmflexnetserver:

```
sudo cp -rv opt/flexnetserver /opt/
```

## Step 3:

```
sudo ln -s /lib/x86_64-linux-gnu/libcrypto.so.1.0.0 /usr/autodesk/maya2017/lib/libcrypto.so.10
sudo ln -s /lib/x86_64-linux-gnu/libssl.so.1.0.0 /usr/autodesk/maya2017/lib/libssl.so.10
```

```
ln -s /usr/lib/x86\_64-linux-gnu/libtiff.so.5 /usr/lib/libtiff.so.3
```

## Step 4:

Install additional dependencies

```
sudo apt-get install csh libjpeg62 libssl-dev libfam0 libcurl3
```

Install fonts

```
sudo apt-get install -y ttf-liberation xfonts-100dpi xfonts-75dpi
```

## Step 5:

Maya will error saying libXp.so.6. Unfortunatly this is nolonger packaged in ubuntu or mint, so you need to download libXp6 from <http://packages.ubuntu.com/trusty/amd64/libxp6/download> and install it

## Step 6:

```
sudo ln -s /usr/lib/x86\_64-linux-gnu/libtiff.so.5 /usr/lib/libtiff.so.3
sudo ln -s /usr/lib/x86\_64-linux-gnu/libjpeg.so.8.0.2 /usr/lib/libjpeg.so.62
```

## Step 7:

```
sudo mkdir /usr/tmp
sudo chmod 777 /usr/tmp
```

## Step 8:

Fix segmentation fault error

```
echo "MAYA\_DISABLE\_CIP=1" > ~/maya/2017/Maya.env
```