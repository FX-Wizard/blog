+++
title = "How to share your Thinkbox Deadline Repository with Samba"
date = "2023-08-11T10:48:34+10:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["LINUX", "DEADLINE", "HOW TO"]
keywords = ["linux", "samba", "deadline", "thinkbox"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

If you want to use direct connect to connect your Deadline Workers to the Repository, or if you want to make it easy to add scripts and plug-ins to Deadline, you need a file share. But how to set one up? The simple free solution is Samba. Here is how to get it set up on your Deadline server.

**What is Samba?**

Put simply, [Samba](https://www.samba.org/samba/what_is_samba.html) is a file server for Linux.

Sharing the Deadline Repository with Samba will allow Windows and Linux clients to connect directly to the repository. It will also make it easier to add and update custom scripts and plug-ins to Deadline.

### Install Samba

Ubuntu and Debian
```bash
sudo apt update
sudo apt install samba -y
```

Rocky and RedHat
```bash
sudo dnf install samba samba-common samba-client  -y
```

**Check if the service is running**

```bash
systemctl status smbd.service | grep Active
```

If the above command says `active (running)` congratulations Samba is installed and working


**Firewall rules**

If your firewall is enabled you will need to allow Samba traffic through.

Ubuntu and Debian
```bash
sudo ufw allow samba
```

Rocky and RedHat
```bash
sudo firewall-cmd --permanent --add-service=samba
sudo firewall-cmd --reload
```

### Share the Repository

With Samba now installed we need to tell it to share the Deadline Repository folder. To do that we need to modify Samba's config file `smb.conf`.

To start, backup the original config file

```bash
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.old
```

With your favourite text editor, edit the config file.

For example.
```
sudo nano /etc/samba/smb.conf
```

Copy the following into the smb.conf file.

This first section contains the basic configuration needed to make Samba work and does not need any changes, so you can copy it into the smb.conf file without changing anything.

```ini
[global]
   workgroup = WORKGROUP
   server string = %h server (Samba, Ubuntu)
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
```

This second section has the configuration needed to share the Deadline Repository directory.

If you have installed the Deadline Repository to the default location you can simply copy and paste this into your smb.conf file
However if you have put the Repository in a different location you need to change the `path =` variable to the location where you have installed the Repository

```ini
[deadline]
   comment = Deadline Repository
   path = /opt/Thinkbox/DeadlineRepository10
   read only = no
   browsable = yes
   guest ok = yes
   writeable = yes
   create mask = 0775
   directory mask = 0775
```

Test the config file for errors
```bash
testparm
```

Restart the Samba service
```bash
sudo systemctl restart smbd.service
```


### Create user account

Our current set-up does not have any security and lets anyone connect to the shared folder with full read/write access.

To add some basic user authentication we need to create a Samba user account and set a password

```
sudo smbpasswd -a username
```

For example
```bash
sudo smbpasswd -a deadline
```

Remove the line `guest ok = yes` from the smb.conf file so it looks like this

```ini
[deadline]
   comment = Deadline Repository
   path = /opt/Thinkbox/DeadlineRepository10
   read only = no
   browsable = yes
   writeable = yes
   create mask = 0775
   directory mask = 0775
```

Restart the Samba service so it loads the new configuration file
```bash
sudo systemctl restart smbd.service
```

Now when you try to access the Deadline Repository shared folder it will require a username and password.

*Note:*
It's also possible to have Samba authenticate users against Active Directory, however that is beyond the scope of this simple guide.

### Sharing with Deadline clients

Now that the repository file share is up and running it's time to mount it on the clients.

You will need the IP address or DNS name of the Deadline Repository server. The below assumes your server is called `deadline`.

For Linux

```bash
mount -t cifs //deadline/deadline /mnt/deadline
```

> Note: You need to make sure the directory you are mounting to exists.
> For example if you want to mount the share to `/mnt/deadline`
> ```bash
> sudo mkdir /mnt/deadline
> ```

To autmoatically mount the drive on start

```bash
cat << EOF >> /etc/fstab
# Deadline Repository network share
//deadline/deadline /mnt/deadline cifs rw,auto,noperm,user=deadline,password=<your password>,file_mode=0777,dir_mode=0777 1 1
EOF
mount /mnt/deadline
```

If you have set a username and password you'll need to specify it when mounting the shared folder.

```bash
mount -t cifs //deadline/deadline /mnt/deadline -o username=<your username>
```

> **Note:**
> If you get this error
> ```
> mount: /mnt/deadline: bad option; for several filesystems (e.g. nfs, cifs) you might need a /sbin/mount.<type> helper program.
> ```
> You might not have the cifs-utils installed.
> Heres how to install it on Debian/Ubuntu.
> ```bash
> sudo apt install cifs-utils
> ```

For Windows using cmd
```cmd
net use R: \\deadline\deadline
```

If you have specifed a username/password

```cmd
net use R: \\deadline\deadline /user:<your username>
```

All that's left to do is connect the Deadline client to the repository, [the instructions for which you can find here.](https://docs.thinkboxsoftware.com/products/deadline/10.3/1_User%20Manual/manual/changing-repository.html)