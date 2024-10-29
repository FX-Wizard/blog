+++
title = "Ubuntu 24.04 3D Workstation in AWS"
date = "2024-10-29T06:25:05Z"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["AWS", "LINUX", "UBUNTU"]
keywords = ["aws", "linux", "ubuntu"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

An updated version of my old Ubuntu workstation on AWS guide, focusing on Ubuntu 24.04 running on a g6 EC2 instance.

## Prerequisites

This guide assumes you already have an EC2 instance with an Nvidia GPU running Ubuntu 24.04.

---

## Getting started

Update the system.

```bash
sudo apt update
sudo apt upgrade -y
```

*Note: It's a good idea to give the system a reboot after the update so it can load any kernel updates.*

---

## Desktop Environment Installation

Next step is installing a desktop environment. This is the hardest part of the set up, not because it's difficult but you have to choose which desktop environment you want to use.

### Installing GDM

Amazon DCV only supports GDM so regardless of your chosen desktop environment GDM needs to be used for login.

```bash
sudo apt install gdm3 -y
sudo systemctl enable gdm
```

Currently Nvidia and Amazon DCV don't fully support Wayland so we need to disable it in GDM.

```bash
sudo sed '/WaylandEnable=false/s/^#//' -i /etc/gdm3/custom.conf
```

### Installing a Desktop Environment

**Gnome**
```bash
sudo apt install -y ubuntu-desktop
```

**MATE**
```bash
sudo apt install -y ubuntu-mate-desktop
```

Make sure to select gdm3 as the default display manager.

**KDE**
```bash
sudo apt install kubuntu-desktop -y
```

---

## Nvidia drivers

Disable the Nouveau open source driver so it can't conflict with the proprietary Nvidia driver.

```bash
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo 'omit_drivers+=" nouveau "' | sudo tee /etc/dracut.conf.d/blacklist-nouveau.conf
```

### Install dependencies

Install DKMS so the driver doesn't break every time the kernel is updated.
```bash
sudo apt install -y dkms
```

Some 3D graphics software requires libglvnd to work, so lets install it so it can be enabled in the driver.

```bash
sudo apt install pkg-config libglvnd-dev -y
```

Install build tools + headers to compile the driver kennel module
```bash
sudo apt-get install -y gcc make linux-headers-$(uname -r)
```

### Download Nvidia driver

There are a few options for getting the Nvidia driver. You can download directly from Nvidia's website, however this might not always work depending on the instance type you are using. The best option is to get the driver direct from AWS, that way you know it will work correctly with EC2.

[Here is a guide to downloading the driver direct from AWS](/posts/nvidia-driver-linux-ec2-download)

### Install Nvidia driver
```bash
chmod u+x NVIDIA*.run
sudo ./NVIDIA*.run -s --dkms --install-libglvnd
```

Enable multiple-monitor support.
```bash
sudo nvidia-xconfig --preserve-busid --enable-all-gpus --connected-monitor=DFP-0,DFP-1,DFP-2,DFP-3
```

---

## Amazon DCV Installation

*Note: Amaozon DCV was formally known as Nice DCV and may still have the old name in places*

First we need to download Amazon DCV by going to the https://www.amazondcv.com/ and grabbing the download URL.

Then use `wget` command to download Amazon DCV from the URL we got from the website like below.

```bash
wget https://d1uj6qtbmh3dt5.cloudfront.net/2024.0/Servers/nice-dcv-2024.0-17979-ubuntu2404-x86_64.tgz
```

Extract the tar file.

```bash
tar zxvf nice-dcv-*.tgz
```

Change into the extracted directory using `cd`.

Install all the components for the Amazon DCV server.

```bash
sudo apt install -y ./nice-*.deb
```

Install USB drivers.

```bash
sudo dcvusbdriverinstaller
```

When asked if you want to install the USB kernel module choose yes by pressing y then enter.

```
Do you want to install the kernel module and enable USB remotization? [y|n]
```

Make sure to use DKMS pressing y then enter.

```
Do you want to use DKMS to install the kernel module? [y|n]
```

Install pulseaudio-utils for audio support.

```bash
sudo apt install pulseaudio-utils
```

If the firewall is enabled add the following rules to allow Amazon DCV. 

```bash
sudo ufw allow 8443/tcp
sudo ufw allow 8443/udp
```

Autostart DCV console session as current user.

```bash
sudo sed '/create-session = true/s/^#//' -i /etc/dcv/dcv.conf
sudo sed -i "s/#owner = \"\"/owner = \"$USER\"/" /etc/dcv/dcv.conf
```

Add the dcv user to the video group.
```bash
sudo usermod -aG video dcv
```

Start the Amazon DCV Server service.
```bash
sudo systemctl enable dcvserver
```

Set the system to boot into graphical mode.

```bash
sudo systemctl set-default graphical.target
```

---

## Login

Don't forget to set a password so you can log in.

```bash
sudo passwd $USER
```

Give the system one final reboot. Once started back up you can remote in using Amazon DCV and use the system.

---

## Optional extras

**App store stuff**

I find Ubuntu's snaps to be slow and there are less options for apps. So I like to add flatpaks for more apps!

```bash
sudo apt install flatpak -y
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
sudo apt install gnome-software -y
sudo apt install gnome-software-plugin-flatpak -y
```