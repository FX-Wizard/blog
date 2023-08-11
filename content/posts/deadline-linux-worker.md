+++
title = "Thinkbox Deadline Linux Worker Installation"
date = "2023-04-03T10:06:15+10:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["thinkbox", "deadline", "render farm"]
keywords = ["Deadline", "Render"]
description = "How to install Deadline Client on Linux using CLI"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

## Getting started

**Which Linux Distro?**

This example uses [Rocky 8](https://rockylinux.org/), however any major distro should work.
Rocky Linux is officially supported by industry standard software such as Autodesk Maya making it an ideal choice for rendering.

**Preparation**

Run software updates

```bash
sudo dnf update -y
```


## Download Deadline

http://downloads.thinkboxsoftware.com/

```bash
curl -O https://thinkbox-installers.s3.us-west-2.amazonaws.com/Releases/Deadline/10.3/1_10.3.0.9/Deadline-10.3.0.9-linux-installers.tar
```

**Extract the installer**

```bash
tar xvf Deadline*.tar
```

## Copy SSL certificate to worker

During the Deadline Repository installation a certificate was created. Copy this certificate to the worker.

The default location of the certificate on the Repository is:
```sh
/opt/Thinkbox/Deadline10/certs/Deadline10RemoteClient.pfx
```

`scp` is one method you could use to copy the certificate. For example.
```bash
scp ubuntu@deadline:/opt/Thinkbox/Deadline10/certs/Deadline10RemoteClient.pfx rocky@worker:/home/rocky/
```

## Install Worker

Run the installer
```bash
sudo ./DeadlineClient-*-linux-x64-installer.run
```


```
----------------------------------------------------------------------------
Welcome to the Deadline Client 10.2.1.0 Setup Wizard.

----------------------------------------------------------------------------
Please read the following License Agreement. You must accept the terms of this 
agreement before continuing with the installation.

Press [Enter] to continue:
```
Accept the EULA to continue.


```
----------------------------------------------------------------------------
Client Installation Directory

Please specify the directory where Deadline Client will be installed.

Installation Directory [/opt/Thinkbox/Deadline10]:
```
This example uses the default location.

*press enter*


```
Set full read/write access for files for all users [y/N]: 
```
*press enter*


```
----------------------------------------------------------------------------
Select Installation Type

Please pick what to install

This page allows to choose between installation of various setups each of which require client setup.

[1] Client: Installs all components which are needed for a Deadline client.
[2] Remote Connection Server: Select this option if you are installing a server. For client installation RCS is not needed.
[3] Deadline Web Service: The Deadline Web Service application is a command line application for the Deadline render farm management system. It allows you to get query information from Deadline over an Internet connection.
Please choose an option [1] : 
```
This is going to be a render node that runs Deadline Worker, so we want to select 1 to install the client.

*press enter*


```
----------------------------------------------------------------------------
Deadline Setup

Connection Type

The Repository Connection Type that Deadline will connect to.

Connection Type

[1] [Recommended] Remote Connection Server: Select this option if you are connecting to a Remote Connection Server.
[2] Direct Connection: Select this option if you are connecting to a repository using the file system.
Please choose an option [1] : 
```
In this example we are using the Remote Connection Server (RCS) to connect to the Repository.

If you are connecting directly to the Repository via a network file share, select option 2.

*press enter*

```
----------------------------------------------------------------------------
Deadline Setup

The hostname or IP address and the port used to communicate with a remote 
server.

Server Address [127.0.0.1:8080]:
```
Enter the address of your deadline repository

If using ssl the default port is 4433

For example:

*deadline:4433*



```
----------------------------------------------------------------------------
Deadline Setup

The RCS Client TLS certificate (Deadline10RemoteClient.pfx). By default, it can 
be found in the Deadline Client installation in the "certs" folder (on the 
machine where the Remote Connection Server setup was performed). Leave blank if 
the Remote Connection Server does not require client authentication.

This is not the same as the certificate that is used for the Deadline Database 
(Deadline10Client.pfx).

RCS TLS Certificate []: 
```
This is the location of the certificate we copied from the repository [earlier in this step](#copy-ssl-certificate-to-worker).

For example:
*/home/rocky/Deadline10RemoteClient.pfx*



```
The password for the client certificate specified above, if required. Leave 
blank if the PFX file is not encrypted, and does not require a password.

Certificate Password :
```




```
----------------------------------------------------------------------------
Deadline Launcher Setup

The Launcher allows for remote communication between Deadline Clients.


Launch Worker When Launcher Starts [Y/n]: 
```
*press y then enter*


```
Install Launcher As A Daemon [y/N]: 
```

If installing the worker on an Artists workstation its best to select `N` because the user logged in will not be able to easily manage the worker.

However if this is on a dedicated render node select `Y` so the worker starts after the render node has booted.

*press y then enter*



```
----------------------------------------------------------------------------
The user to run the Launcher daemon as.

User Name []: 
```

This is the user the Deadline Worker will run as. It can be any existing user, however *root* is not recommended. 

In this example the user *rocky* is used because it already exists.

*type rocky then press enter*


```
Running Deadline Launcher as a daemon requires an open file descriptor limit (ulimit -n) to be set to the recommended value of 200000.

You can change this limit in /etc/systemd/system/deadline10launcher.service after the installation has finished, but Deadline Launcher will have to be restarted before it recognizes the change. Refer to the Deadline installation documentation for more information on resource limits.

Do you wish to continue? [Y/n]: 
```
*press enter*



```
----------------------------------------------------------------------------
Block Auto Update Override

Auto upgrade is a feature of Deadline that permits Deadline to upgrade itself based on files in the repository. Although useful, we recommend customers block this feature unless they need it as this increases security.

[1] Block auto upgrade via a secure setting. (Recommended): This option disables auto upgrade on this install, ignoring any auto upgrade settings in the repository.
[2] Use the repository settings to determine if auto upgrade is enabled.: This will use the repository (and the local override setting, if any) to determine if auto upgrade is enabled or disabled.
Please choose an option [1] : 
```
If running Deadline on-prem this can be a very useful way to ensure the client software stays up to date on all the render nodes.
However if you are using another method to manage software upgrades or running on the cloud using golden images, such as AWS AMIs, its best to leave this option disabled.

*press enter*

```
----------------------------------------------------------------------------
Setup is now ready to begin installing Dead line Client on your computer.

Do you want to continue? [Y/n]: 
```

*press enter*


```
----------------------------------------------------------------------------
Please wait while Setup installs Deadline Client on your computer.

 Installing
 0% ______________ 50% ______________ 100%
 #########################################

----------------------------------------------------------------------------
Setup has finished installing Deadline Client on your computer.

```

Once installation is complete the Deadline Launcher should start automatically and if enabled in the installation the Deadline Worker will be automatically launched on Launcher start.


### Checking connection to Repository

To check if the Worker has sucessfully connected to the Repository via CLI

```bash
/opt/Thinkbox/Deadline10/bin/deadlinecommand -GetFarmStatistics
```

More info on using deadlinecommand can be found [here](https://docs.thinkboxsoftware.com/products/deadline/10.3/1_User%20Manual/manual/command.html)
