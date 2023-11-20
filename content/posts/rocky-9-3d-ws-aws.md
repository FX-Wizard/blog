+++
title = "Rocky 9 3D Workstation on AWS"
date = "2023-09-11T15:15:49+10:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["AWS", "LINUX"]
keywords = ["aws", "linux", "rocky 9", "workstation"]
description = "3D workstation in the AWS cloud with Rocky 9 Linux"
showFullContent = false
readingTime = false
hideComments = false
toc = true
color = "orange" #color from the theme settings
+++

The objective is simple, build a 3D workstation in the cloud with Rocky 9 Linux for running application like Blender, Maya, Nuke, etc...

If you prefer Ubuntu I have another guide [here](/posts/ubuntu-22.04-3d-ws-aws) for that distro.

---

## Prerequisites

This guide assumes you already have an EC2 instance with an Nvidia GPU running Rocky 9.

---

## Getting Started

Update the system.

```bash
sudo dnf update -y
```

Install EPEL Repository. This will make it easy to install extra packages like the AWS CLI.

```bash
sudo dnf install epel-release -y
sudo dnf config-manager --set-enabled crb
sudo crb enable
```


AWS CLI

If you have installed the EPEL repository you can simply install with `dnf`.

```bash
sudo dnf install awscli -y
```

Or to install manually.

```bash
sudo dnf install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Cleanup install files from manual installation.
```bash
rm -rf aws*
```

### Install AWS SSM Agent (optional)

Systems Manager (SSM) is a handy service for remote patching/monitoring/administrating. It needs an agent installed to work its magic on the remote system.

```bash
sudo dnf install -y https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm
```

---

## Desktop Environment Installation

### Installing GDM

Install GDM for login. Nice DCV only works with GDM at this stage.

```bash
sudo dnf install -y gdm
sudo systemctl enable gdm.service
```

Nvidia and Nice DCV currently don't support Wayland so it needs to be disabled.

```bash
sudo sed '/WaylandEnable=false/s/^#//' -i /etc/gdm3/custom.conf
```

### Installing a Desktop Environment

Now you have a choice to make, which desktop environment to install...

**Gnome**

```bash
sudo dnf groupinstall "Workstation"
```

**KDE**

```bash
sudo dnf groupinstall "KDE Plasma Workspaces"
```
KDE will install its own desktop manager, it needs to be disabled so GDM can be used instead.

```bash
sudo systemctl disable --now sddm.service
```

---

## Install Nvidia Drivers

### Disable Nouveau

Disable the Nouveau open source driver so it can't confict with the proprietary Nvidia driver.
```bash
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo 'omit_drivers+=" nouveau "' | sudo tee /etc/dracut.conf.d/blacklist-nouveau.conf
```

*OPTIONAL: Sometimes the desktop environment installation will also install Nouveau with it, so you'll need to regenerate the initramfs to ensure Nouveau is not loaded by the kernel.*
```bash
sudo dracut --regenerate-all --force
sudo depmod -a
```

Install dependencies for the driver.

```bash
sudo dnf install -y make gcc elfutils-libelf-devel libglvnd-devel kernel-devel-$(uname -r)
```

*NOTE: If you get an error such as `Error: Unable to find a match: kernel-devel` you might need to reboot to load the updated kernel.*

### Download the Nvidia driver 

There are a few options for getting the Nvidia driver. You can download directly from Nvidia's website, however this might not always work depending on the instance type you are using. The best option is to get the driver direct from AWS, that way you know it will work correctly with EC2.

[Here is a guide to downloading the driver direct from AWS](/posts/nvidia-driver-linux-ec2-download)

### Install Nvidia driver

<!-- TODO: explain a bit more about what this command is doing -->

```bash
chmod +x NVIDIA-Linux-x86_64*.run
sudo /bin/sh ./NVIDIA-Linux-x86_64*.run -s --dkms --install-libglvnd
```

Disable GSP.
```bash
sudo touch /etc/modprobe.d/nvidia.conf
echo "options nvidia NVreg_EnableGpuFirmware=0" | sudo tee --append /etc/modprobe.d/nvidia.conf
```

Update xorg.conf to enable multiple monitor support.
```bash
sudo nvidia-xconfig --preserve-busid --enable-all-gpus --connected-monitor=DFP-0,DFP-1,DFP-2,DFP-3
```

Test if its working.

```bash
nvidia-smi -q | head
```

---

## Nice DCV Installation

First we need to download Nice DCV by going to the [Nice DCV website](https://download.nice-dcv.com/) and grabbing the download URL.

Then use `curl` command to download Nice DCV from the URL we got from the website like below.

```bash
curl -O https://d1uj6qtbmh3dt5.cloudfront.net/2023.1/Servers/nice-dcv-2023.1-16220-el9-x86_64.tgz
```

Extract the tar file.

```bash
tar zxvf nice-dcv*.tgz
```

Change into the extracted directory using `cd`.

Install all the components for the Nice DCV server.

```bash
sudo dnf install ./nice-*.rpm -y
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

Allow firewall

```bash
sudo firewall-cmd --zone=public --permanent --add-port=8443/tcp
sudo firewall-cmd --zone=public --permanent --add-port=8443/udp
sudo systemctl reload firewalld
```

Autostart DCV console session as current user.

```bash
sudo sed '/create-session = true/s/^#//' -i /etc/dcv/dcv.conf
sudo sed -i "s/#owner = \"\"/owner = \"$USER\"/" /etc/dcv/dcv.conf
```

Enable Nice DCV service.

```bash
sudo systemctl enable dcvserver.service
```

Set the system to boot into graphical mode.

```bash
sudo systemctl set-default graphical.target
```

---

## Login

Set a password for the user so you can login with DCV.

For example
```bash
sudo passwd $USER
```

Give it one final reboot!

```bash
sudo reboot
```

Once the system comes back up its ready to use!

---

## Clean up

Don't forget to delete the Nice-DCV and Nvidia driver installers.

---

## Troubleshooting

**- Connection refused by DCV**

If firewalld service is running but has not added the rules to allow Nice DCV.

This usually happens if you get the error `FirewallD is not running` when adding the firewall rules.

To fix this just add the firewall rules again.

**- Nice DCV shows a blank blue screen with a spinning wheel forever**

This usually indicated Nice DCV is working but the graphical session is not.

The most likely culprit is the Nouveau driver is running instead of the Nvidia proprietary driver. You can check if this is the case by running this command.

```bash
lsmod | grep nouveau
```

If Nouveau is running try repeating the [steps to disable Nouveau](#disable-nouveau) including running the dracut commands, then reboot the system.

Once the system has reboot check to see if the Nvidia driver is loaded by running.

```bash
lsmod | grep -i nvidia
```
