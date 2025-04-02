+++
title = "Maya 2016/17 Install On AWS"
date = "2017-06-02T12:57:17+11:00"
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

Use Centos7 as the base AMI

## Maya installation

`rpm -ivh *.rpm`

```
sudo yum install -y xorg-x11-fonts-ISO8859-1-100dpi xorg-x11-fonts-ISO8859-1-75dpi xorg-x11-fonts-100dpi xorg-x11-fonts-75dpi  
sudo yum install -y mesa-libGLw mesa-libGLU csh tcsh libXp libXpm libXp-devel gamin audiofile audiofile-devel e2fsprogs-libs  
sudo yum install -y compat-libtiff3.x86_64  
sudo yum install -y libpng12.x86_64
```

this stuff would be here if there was a gui like gnome on the os

`sudo yum install -y gstreamer-plugins-base-0.10.36-10.el7.x86_64`

**extra stuff for 2017:**

```
yum install libx**
yum install libX**
yum install libpng12.so.0
yum libpng12
```

edit `/etc/yum.conf`

add line

`exclude=*.i386 *.i586 *.i686`

```
yum install pulseaudio
ln -s /usr/lib64/libpulse.so.0 /usr/autodesk/maya2017/lib/libpulse.so.0
```

## License stuff!

### Maya 2016

```
export LD\_LIBRARY\_PATH=/opt/Autodesk/Adlm/R11/lib64/
```

```
/usr/autodesk/maya2016/bin/adlmreg -i N 657H1 657H1 2016.0.0.F 393-83854580 /var/opt/Autodesk/Adlm/Maya2016/MayaConfig.pit  
```

Yes you have to put the product key in twice or it will say invalid number of arguments

from the folder where you untared the installer

```
/usr/autodesk/maya2016/bin/licensechooser /usr/autodesk/maya2016 network 657H1 maya
```

### Maya 2017

```
export LD\_LIBRARY\_PATH=/opt/Autodesk/Adlm/R12/lib64/
```

```
/usr/autodesk/maya2017/bin/adlmreg -i N 657I1 657I1 2017.0.0.I 393-83854580 /var/opt/Autodesk/Adlm/Maya2017/MayaConfig.pit
```

Yes you have to put the product key in twice or it will say invalid number of arguments

from the folder where you untared the installer

```
/usr/autodesk/maya2017/bin/licensechooser /usr/autodesk/maya2017 network 657I1 maya
```

### Maya 2018

```
export LD\_LIBRARY\_PATH=/opt/Autodesk/Adlm/R14/lib64/
```

```
/usr/autodesk/maya2018/bin/adlmreg -i N 657J1 657J1 2018.0.0.F 393-83854580 /var/opt/Autodesk/Adlm/Maya2018/MayaConfig.pit  
```

Yes you have to put the product key in twice or it will say invalid number of arguments

from the folder where you untared the installer

```
/usr/autodesk/maya2018/bin/licensechooser /usr/autodesk/maya2018 network 657J1 maya
```

(if adlmreg is missing libadlmutil.so just copy adlmreg to /opt/Autodesk/Adlm/R14/lib64/ where the lib file is)

## Vray for Maya

install the vray stuff

```
sudo ./vray_adv_36003_maya2018_linux_x64 -gui=0
```

### Vray Licence

```
export VRAY_AUTH_CLIENT_FILE_PATH=path/to/file/vrlclient.xml
```