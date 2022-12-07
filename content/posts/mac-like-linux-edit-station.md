+++
title = "Mac Like Linux Edit Station"
date = "2022-12-01T23:15:04+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
draft = true
tags = ["tech", "aws", "linux"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "red" #color from the theme settings
+++


# Editors cloud dreamstation

Software
- Davinci Resolve
- Kdenlive


## Base system setup 
Ubuntu 22.04 base

update the system
```bash
sudo apt get update && sudo apt get upgrade -y
```
reboot to load the latest kernel updates


Install AWS CLI
```bash
sudo apt install awscli
sudo apt install nfs-common cifs-utils
```

## Nvidia drivers

Install DKMS to prevent headaches when updating

```bash
sudo apt install dkms
```

*need to add IAM role for read only access to s3*

```bash
aws s3 cp --recursive s3://ec2-linux-nvidia-drivers/latest/ .
chmod u+x NVIDIA*.run
sudo ./NVIDIA*.run
```

Enable multi-montior support
```bash
sudo nvidia-xconfig --preserve-busid --enable-all-gpus --connected-monitor=DFP-0,DFP-1,DFP-2,DFP-3
```

## Desktop environment

Install Pantheon from Elemtary OS

```bash
sudo add-apt-repository ppa:elementary-os/stable
sudo apt-get update
sudo apt-get install elementary-desktop
```

Make sure to select gdm3 because NiceDCV does not like to work with anything else. (attach screenshots)

Set defaul session to Pantheon

```bash
sudo sed 's/Session=/Session=pantheon/' /var/lib/AccountsService/users/cinnamon
```

## Pixel streaming with NiceDCV

Download NiceDCV
```bash
wget https://d1uj6qtbmh3dt5.cloudfront.net/2022.2/Servers/nice-dcv-2022.2-13907-ubuntu2204-x86_64.tgz
```

extract 

```bash
tar zxvf *nice-dcv*
```

open extraced folder

install everything in the folder

```bash
sudo apt install -y ./*.deb
```

autostart DCV console session (not working!)
```bash
sudo sed 's/#create-session = true/create-session = true/' /etc/dcv/dcv.conf
sudo sed 's/#owner = ""/owner="ubuntu"/' /etc/dcv/dcv.conf
```

Add the dcv user to the video group
```bash
sudo usermod -aG video dcv
````

Enable the DCV service
```bash
sudo systemctl enable --now dcvserver
```

Switch Linux to graphical mode
```bash
sudo systemctl set-default graphical.target
sudo systemctl isolate graphical.target
```


## Last Steps

Dont forget to set a password
```bash
sudo passwd ubuntu
```

reboot 

log in to DCV



# Software setup

## Good

### Installing DaVinci Resolve
Resolve needs some extra packages installed
```bash
sudo apt install libxcb-composite0 libxcb-cursor0
```

Welcome to resolve page might be blank on first start


### Ulauncher
spotlight like search on Linux

```bash
sudo add-apt-repository ppa:agornostal/ulauncher
sudo apt update
sudo apt install ulauncher -y
```