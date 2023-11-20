+++
title = "Updating Nvidia Driver on Amazon Workspaces Ubuntu"
date = "2023-10-20T21:48:27+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["AWS", "WORKSPACES", "UBUNTU", "NVIDIA"]
keywords = ["amazon workspaces", "aws", "linux", "ubuntu"]
description = "How to upgrade the Nvidia drivers to the latest version on Ubuntu graphical Amazon Workspaces."
showFullContent = false
readingTime = false
hideComments = false
toc = true
color = "orange" #color from the theme settings
+++

When AWS added support for Ubuntu graphics instances for Amazon Workspaces I was excited at the possibilities for running 3D apps like Unreal 5 and Blender. However once I got one up and running I was surprised to see the Nvidia graphics drivers were woefully out of date. So here is a guide to fix that and get the latest Nvidia drivers on Workspaces.

## Prerequisites

Nvidia driver for Linux does not like to install while a X11 is running a graphical interface. You'll need a way to remotely access the workspace through SSH. The easiest way to do this is with a jump box (sometimes known as a bastion host).

Here is a guide to setting one up if you don't have one.

https://aws.amazon.com/solutions/implementations/linux-bastion/

The jump box will need.
- security group to allow SSH outgoing to the VPC your Amazon Workspaces are deployed in.
- IAM role to download Nvidia driver. (The how to is covered in this guide).
- Ideally running Ubuntu 22.04.

## Getting started

First connect to the workspace normally to make sure it's working correctly.

Open a terminal and update the workspace because it needs more than just a driver update out of the box.

```bash
sudo apt update
sudo apt upgrade -y
```

After the update make sure to reboot so any kernel updates can be applied.

If your workspace kicks you off after the update and won't let you back in, don't panic! Go into the console and manually reboot and you should be able to get back in.

---

## SSH access to workspace

The current state of Nvidia on Linux is a mess. To install the driver the GUI can't be loaded, but that means we cannot access the Workspace anymore because there is no GUI. To work around this we need another workspace, or an EC2 instance in the same subnet to SSH into the Workspace and run the upgrade.

### Configure SSH

To connect with SSH we either need to create a new key, or allow password authentication. 

To keep things simple for this instruction I'm going with password authentication. I'm not allowing SSH from the internet and I'm planning to disable this setting after, so password authentication is secure enough for this purpose.

To enable password authentication in SSH
```bash
sudo sed -i "s/PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config
sudo systemctl reload sshd
```

Get the IP address of the Workspace. Make sure to write this down so you can get in later.
```bash
ip address | grep inet
```

---

## Nvidia Driver Download

The AWS specific Nvidia driver lives in an S3 bucket. We need to give the jump box access to that bucket to download the driver.

[Here is a guide to downloading the driver](/posts/nvidia-driver-linux-ec2-download). Follow it and once the driver is downloaded come back here and we can continue with installing it onto the workspace.

## Copy Driver to Workspace

Copy driver to workspace via [scp](https://linux.die.net/man/1/scp).

You'll need to specify the username to log into the workspace, and the IP address of the workspace.

```bash
scp NVIDIA-Linux-x86_64-*-aws.run <username>@<workspace ip>:/tmp/
```

---

## Nvidia Driver Installation

From the jump box ssh into the workspace.

```bash
ssh <user>@<workspace ip>
```

Once connected to the workspace via SSH.

Make sure the build tools and kernel headers are installed.

```bash
sudo apt update
sudo apt-get install -y gcc make linux-headers-$(uname -r)
```

Switch to multi user and reboot because the Nvidia driver wont work if a GUI is running on X.

*Note: From this point forward you wont be able to access the workspace normally with the client until the driver update is complete.*

```bash
sudo systemctl set-default multi-user.target
sudo reboot
```

It should only take a minute to reboot then you can SSH into the workspace again.

Once reconnected to the workspace, open the folder where you copied the driver to. If you are following the guide this is the `/tmp` directory.

```bash
cd /tmp
```

Make the installer file executable.
```bash
sudo chmod u+x NVIDIA-Linux*.run
```

Run the driver installer.

```bash
sudo ./NVIDIA-Linux*.run 
```

I had an odd error about the tmp directory not being executable even though it was. To work around this by making a new folder and adding `--tmpdir nvidia-tmp` when running the installer.

```bash
sudo mkdir /nvidia-tmp
sudo chmod 777 /nvidia-tmp
sudo ./NVIDIA-Linux*.run --tmpdir nvidia-tmp
```

Don't forget to delete the temp folder after install.

Once the installation is complete switch the workspace back to graphical mode and reboot.

```bash
sudo systemctl set-default graphical.target
sudo reboot
```

After the workspace has rebooted you should be able to connect to it again using the Workspaces client.

You can now use your workspace with the new Nvidia driver installed on it.