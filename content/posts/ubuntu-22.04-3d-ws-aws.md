+++
title = "Ubuntu 22.04 3D Workstation in AWS"
date = "2023-10-09T10:37:52+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["AWS", "LINUX", "UBUNTU"]
keywords = ["aws", "linux", "ubuntu"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
toc = true
color = "orange" #color from the theme settings
+++

This is a guide to setting up a 3D workstation in the cloud with Ubuntu 22.04 Linux for running application like Blender, Maya, Nuke, etc...

I also have a dedicated guide for Rocky 9 [here](/posts/rocky-9-3d-ws-aws) if Ubuntu is not your thing.

---

## Prerequisites

This guide assumes you already have an EC2 instance with an Nvidia GPU running Ubuntu 20.04 or higher.

---

## Getting started

Update the system.

```bash
sudo apt update
sudo apt upgrade -y
```

*Note: It's a good idea to give the system a reboot after the update so it can load any kernel updates.*

Install the AWS CLI. We will need it later for downloading software from S3.

```bash
sudo apt install -y awscli
```

---

## Desktop Environment Installation

Next step is installing a desktop environment. This is the hardest part of the set up, not because it's difficult but you have to choose which desktop environment you want to use.

### Installing GDM

Nice DCV only supports GDM so regardless of your chosen desktop environment GDM needs to be used for login.

```bash
sudo apt install gdm3 -y
sudo systemctl enable gdm
```

Currently Nvidia and Nice DCV don't fully support Wayland so we need to disable it in GDM.

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

**Others**

There are many other great desktop enviornments out there so don't be affraid to experiment!

[Here is a guide on installing the Pantheon desktop enviornment from Elementary OS](/posts/mac-like-linux-workstation). If you are looking for a more Apple Mac like experience on Linux give it a try.

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
sudo apt install -y libglvnd0
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

## Nice DCV Installation

First we need to download Nice DCV by going to the [Nice DCV website](https://download.nice-dcv.com/) and grabbing the download URL.

Then use `wget` command to download Nice DCV from the URL we got from the website like below.

```bash
wget https://d1uj6qtbmh3dt5.cloudfront.net/2023.1/Servers/nice-dcv-2023.1-16220-ubuntu2204-x86_64.tgz
```

Extract the tar file.

```bash
tar zxvf nice-dcv-*.tgz
```

Change into the extracted directory using `cd`.

Install all the components for the Nice DCV server.

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

If the firewall is enabled add the following rules to allow Nice DCV. 

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

Start the Nice DCV Server service.
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

Give the system one final reboot. Once started back up you can remote in using Nice DCV and use the system.

---

## Optional extras

**App store stuff**

I find Ubuntu's snaps to be slow and there are less options for apps. So I like to add flatpaks for more apps!

```bash
sudo apt install flatpak
sudo apt install gnome-software
sudo apt install gnome-software-plugin-flatpak
```