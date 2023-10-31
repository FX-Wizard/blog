+++
title = "Unreal 5 + AppStream 2.0 + Amazon Linux 2 "
date = "2023-03-09T09:43:26+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["UNREAL", "LINUX", "AMAZON LINUX 2", "APPSTREAM 2.0"]
keywords = ["unreal", "unreal engine", "linux", "amazon linux", "appstream", "aws"]
description = "How to get Unreal Engine 5 running on AppStream 2.0 with Amazon Linux 2"
showFullContent = false
readingTime = false
hideComments = false
color = "orange" #color from the theme settings
+++

The objective is to get Unreal Engine 5 running on AppStream 2.0 with Amazon Linux 2

## Prerequisites

This guide assumes you have a basic knowledge of AppStream 2.0

Launch a new Image builder with Amazon Linux 2 on a g4dn or better instance that has the following network settings.

- Security group that allows incoming SSH (port 22) access
- Internet access so the system can get updates and packages from the internet

## Enable SSH

During this process we will break the Nvidia graphics drivers, so we need SSH to access the system until the drivers are updated.

Install and enable SSH
```bash
sudo yum install -y openssh-server
sudo systemctl enable --now sshd
```

Get the username to use for SSH login
```bash
whoami
```

Set a password to for SSH login
```bash
sudo passwd ImageBuilderAdmin
```

Get the IP address of the instance. Make sure to write this down so you can get in later.
```bash
ip address | grep inet
```

If you try to log in with SSH you may get the bellow error

```
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

This is because password based authentication is currently disabled.

To enable passwords with SSH

```bash
sudo sed -i "s/PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config
sudo systemctl reload sshd
```

#### SSH into the AppStream image

Currently we only know the private IP of the AppStream image, in order to SSH into it we need a bastion host (jumpbox) in the same VPC in order to access the AppStream image. This is relatively easy to set up so I wont go into it here, just get a simple EC2 instance up and running with the needed access and come back when your ready.

---

## Updates

**System updates**
```bash
sudo yum update -y
```

*NOTE: when you update the kernel the graphics drivers will stop working and you wont be able to access the system unless you have SSH set up*

**Kernel updates**
```bash
sudo amazon-linux-extras install kernel-5.15 -y
```

Reboot the system to load the new kernel

---

## Install Dependencies
The Vulkan version in AL2's package repos is out of date and wont work with Unreal. Lucky for us AL2 is basically CentOS 7, So we can just pull the newer packages from the CentOS7 mirrors instead.
```bash
sudo yum install -y http://mirror.centos.org/centos/7/os/x86_64/Packages/vulkan-filesystem-1.1.97.0-1.el7.noarch.rpm
sudo yum install -y http://mirror.centos.org/centos/7/os/x86_64/Packages/vulkan-1.1.97.0-1.el7.x86_64.rpm
sudo yum install -y http://mirror.centos.org/centos/7/os/x86_64/Packages/vulkan-devel-1.1.97.0-1.el7.x86_64.rpm
sudo yum install -y http://mirror.centos.org/centos/7/updates/x86_64/Packages/mesa-vulkan-drivers-18.3.4-12.el7_9.x86_64.rpm
```

Install all the packages needed to compile the driver.
```bash
sudo yum install -y dkms libglvnd libglvnd-devel
sudo yum install -y gcc10 kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```

**Optional**

To prevent `xdg-user-dir` error warning in the Unreal logs. It does not prevent Unreal from working so its optional.
```bash
sudo yum install xdg-user-dirs -y
```

---

## Install Nvidia Drivers

The Nvidia drivers cannot be updated if the kernel already has it loaded. The easy way to make sure the graphics driver are not loaded is to boot the system into a command line only mode.
```bash
sudo systemctl set-default multi-user.target
sudo reboot
```

**Download the drivers**

The recommended way to download the drivers is to get them from AWS. 

*Note: This requires an instance with the AWS CLI installed and an IAM policy that give it access to S3*

```bash
aws s3 cp --recursive s3://ec2-linux-nvidia-drivers/latest/ .
```

For more info on the AWS Nvidia driver:
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-nvidia-driver.html


Make the installer file executable
```bash
sudo chmod u+x NVIDIA-Linux*.run
```

The `CC` environment variable needs to be set to tell the Nvidia installer to use a newer version of GCC. This is because the kernel was updated to a newer version which was compiled with a newer version of GCC.

```bash
sudo CC="gcc10-gcc" ./NVIDIA-Linux*.run
```

Now you can install the Nvidia drivers as normal

Once installation is complete put the system back into graphical mode and reboot

```bash
sudo systemctl set-default graphical.target
sudo reboot
```

**Setting correct clock speeds**

For some silly reason AWS decided not to give us a full GPU and down-clocked them so the GPU is not running at full speed. To fix this we can adjust the clocks back to what they should be.

For G4dn
```bash
sudo nvidia-persistenced
sudo nvidia-smi -ac 5001,1590
```

For G5
```bash
sudo nvidia-persistenced
sudo nvidia-smi -ac 6250,1710
```


---

## Installing Unreal Engine

This is well documented by Epic Games so I wont go in depth on the installation as from this point forward it's like installing Unreal on any other version of Linux.

The prebuilt binaries from Epic Games work well on Aamazon Linux 2 even though they were targeted at Ubuntu.

https://www.unrealengine.com/en-US/linux

> Hint
> AL2 does not have a graphical unzip tool so you can use the cli unzip tool instead
> ```bash
> unzip Linux_Unreal_Engine_*.zip -d ./Unreal
> ```

Compiling from source is also a good option depending on your needs.

https://docs.unrealengine.com/5.1/en-US/building-unreal-engine-from-source/

---

## Clean up

For security don't forget to disable the SSH service

```bash
sudo systemctl disable sshd
```