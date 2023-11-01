+++
title = "Using AD credentials to auto mount network drive on Ubuntu"
date = "2023-11-01T11:48:10+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["LINUX", "UBUNTU"]
keywords = ["active Directory", "samba", "ubuntu", "linux", "autofs", "kerberos"]
description = "Automagically mount a SMB file share on Ubuntu Linux using logged in users AD (Active Directory) credentials."
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

Automagically mount a SMB file share on Ubuntu Linux using logged in users AD (Active Directory) credentials. 

The basic idea is to use the logged in users Kerberos ticket to authenticate the SMB file share with AD and mount it after the user has logged in.

---

## Prerequisites

This guide assumes you have the following already set up

- Active Directory
- SMB File server such as Samba that authenticates to Active Directory
- Ubuntu joined to Active Directory
- Active Directory user account

---

## Installation

[AutoFS](https://help.ubuntu.com/community/Autofs) is the app we will be using to make all the magic happen.

To install it run

```bash
sudo apt update
sudo apt -y install autofs
```

Edit the config file `/etc/auto.master` to add a new mapping config.

```sh
"/- /etc/auto.fileshares --timeout=60"
```

Create new mapping config file `/etc/auto.fileshares` with the details of the mapped drive.

```sh
"/mnt/mountpoint -fstype=cifs,sec=krb5,cruid=$UID ://server/share"
```


You will need to change:

- /mnt/mountpoint - to the path you want to mount the shared drive.
- ://server/share - to the DNS name and share of the file server.

The important parts of the configuration are:

- sec=krb5 - which tells AutoFS to use Kerberos to authenticate
- cruid=$UID - which specifies the ID of the user who's Kerberos ticket will be used, which in this case is the currently logged in user

Once configured restart the autofs service to apply the changes, or just reboot.

```bash
 sudo systemctl restart autofs
```


## Troubleshooting

1. Use DNS name instead of IP Address

2. Check the logs

    ```bash
    sudo cat /var/log/syslog | grep automount
    ```

3. Check Kerberos ticket is valid

    ```bash
    klist
    ```

4. Check its working the ol' fashion way by mounting it manually

    ```bash
    sudo mount -t cifs //server/share /mnt/mountpoint -o username=your_username,sec=krb5
    ```

    > If the manuall mount fails, try using `dmesg` to get the logs

5. smbclient can also test to see if its able to authenticate against the server with Kerberos

    First install it

    ```bash
    sudo apt update
    sudo apt install -y smbclient
    ```

    then run it replacing *server* with the address of your server

    ```bash
    smbclient -k -L server
    ```

6. To get more debug info directly on the terminal

    Edit `/etc/autofs.conf` and set `logging = "debug"` or `logging = "verbose"`

    Then stop autofs and start it from the command line

    ```bash
    sudo systemctl stop autofs
    sudo automount -f -d
    ```
