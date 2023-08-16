+++
title = "Thinkbox Deadline server Linux installation"
date = "2023-02-28T08:17:20+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["thinkbox", "deadline", "render farm"]
keywords = ["Deadline", "Render"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

## Getting started

**Which Linux Distro?**

This example uses [Ubuntu 20.04 LTS](https://ubuntu.com/), however any major distro should work.

> [!Note]
> Newer versions of Linux, like Ubuntu 22.04 no longer ship with libcrypto.so.1.1. This is a requirement for MongoDB 4.4, the database used by the Deadline Repository. If you use a newer Linux you'll need to install the old libcrypto libraries.

**How much CPU & RAM?**

For 20-30 render nodes the server needs about 2 CPU cores and 4GB of ram

If hosting on AWS the t3a.medium instance type works well for about 20 nodes

**Common Terminology**

Here are a few common Deadline terms and their meanings.

Repository and Worker are the 2 main components of a Deadline render farm.

- Repository = Server made up of a file share and database. Manages the render farm.
- Worker = Client that runs the render jobs. Previously called Slave.

Direct connect and Remote Connection Server (RCS) are 2 common terms that frequently appear when working with Deadline.

- Direct Connect = Network share mounted on the Workers that allows the Repository and Workers to directly share files. This is the older method.

- Remote Connection Server (RCS) = HTTP client that sits in between the Repository and Workers and allows them to communicate securely over HTTPS. This is the newer method.


---

## Preparing the server

**Running updates**

```bash
sudo apt update
sudo apt upgrade -y
```

*Make sure to reboot after updates to load any new libs/kernels*


**Setting the servers hostname**

MongoDB needs to know what the servers hostname is to work correctly and it helps to give it a name that makes sense like deadline.
```bash
sudo hostnamectl set-hostname deadline
```

If hosting in AWS make sure to update `/etc/cloud/cloud.cfg` and change `preserve_hostname: false` to `true` or it will change the hostname back on next reboot.

```bash
sudo sed -i 's/preserve_hostname: false/preserve_hostname: true/' /etc/cloud/cloud.cfg
```

---

## Downloading software

This section explains how to download Deadline and MongoDB.

#### MongoDB

Deadline uses Mongodb for its database. As of writing Deadline needs version 4.4.x of Mongodb.

The installer can automatically download MongoDB for supported OS like Ubuntu 20.04, if that's what you're using [skip to the next step](/posts/deadline-linux-server/#deadline). If your OS is unsupported you need to manually download MongoDB and enter the path to where you downloaded it into the installer.

MongoDB downloads page can be found [here](https://www.mongodb.com/try/download/community). The Deadline repository installer wants the tarball package

For example if installing Deadline Repository on Debian 10

```bash
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-debian10-4.4.23.tgz
```

> [!WARNING]
> libcrypto.so.1.1

Mongo 4.4.x depends on libcrypto.so.1.1 which does not ship with the latest Linux distos like Debian 12 or Ubuntu 22.04.

You may get an error like:

`mongod: error while loading shared libraries: libcrypto.so.1.1: cannot open shared object file: No such file or directory`

To fix this either install the older libcrypto library or install an older OS like Debian 10 or Ubuntu 20.04.

#### Deadline

To download Deadline go to the Thinkbox website and grab the download URL.

http://downloads.thinkboxsoftware.com/

```bash
wget https://thinkbox-installers.s3.us-west-2.amazonaws.com/Releases/Deadline/10.3/1_10.3.0.9/Deadline-10.3.0.9-linux-installers.tar
```

To validate the installers follow the instructions [here](https://docs.thinkboxsoftware.com/products/deadline/10.2/1_User%20Manual/manual/validation-linux-installers.html#validation-linux-installers-ref-label).

---
## Prepare repository directory

The Deadline Repository does not need to live on the same server as the Database. Its common practice to put the repository folder itself on a file server, like a NAS, and install the Database elsewhere, like a virtual machine.

For this example the Repository is installed locally so if you want to stick to the guide you can [skip to the next step](/posts/deadline-linux-server/#install-repository).

However if you want to put the repository on a file server continue reading and I'll explain how.

This guide assumes you already have an SMB or NFS file share where you want to host the repository. If not make sure to have that prepared before continuing.

Create directory to mount the file share to. This is where the Deadline Repository will be installed.
```bash
sudo mkdir -p /opt/Thinkbox/DeadlineRepository10
```

> Note
> 
> *Some distros don't come out of the box with the necessary packages to mount network drives installed by default.*
>
> *To install on Debian or Ubuntu*
> ```bash
> # SMB
> sudo apt install -y cifs-utils
> # NFS
> sudo apt install -y nfs-common
> ```
>
> *To install on CentOS or Rocky*
> ```bash
> sudo yum install -y cifs-utils
> ```

Mount the remote folder to that directory
```bash
sudo mount -t nfs <ip-address>:/share/dir /opt/Thinkbox/DeadlineRepository10
```

Create entry in `/etc/fstab` so the folder automounts on start up
```bash
<ip-address>:/share/dir /opt/Thinkbox/DeadlineRepository10 nfs nfsvers=4.1 0 1
```

---
## Install Repository

**Extra dependencies**
If using SSL/TLS certificates the installer needs bunzip2 to be installed

To install it run 

```bash
sudo apt install -y bzip2
```

**Extract the installer**

```bash
tar xvf Deadline*.tar
```

Run the installer
```bash
sudo ./DeadlineRepository-*-installer.run
```

The installer will take a moment to extract and start.

```
----------------------------------------------------------------------------
Welcome to the Deadline Repository 10.3.0.9 Setup Wizard.

----------------------------------------------------------------------------
Please read the following License Agreement. You must accept the terms of this 
agreement before continuing with the installation.

Press [Enter] to continue:
By downloading or using this software, you agree to the AWS Customer Agreement 
(https://aws.amazon.com/agreement/) and AWS Intellectual Property License 
(https://aws.amazon.com/legal/aws-ip-license-terms/). You acknowledge that this 
software is AWS Content as defined in those Agreements.

Press [Enter] to continue:

Do you accept this license? [y/n]: y
```

Accept the EULA to continue.

```
----------------------------------------------------------------------------
Repository Installation Directory

After the installation has finished, you must share this directory with full 
read and write permissions so that the Deadline Clients can access the 
Repository. Refer to the Deadline installation documentation for more 
information on sharing the Repository directory.

Repository Directory [/opt/Thinkbox/DeadlineRepository10]: 
```
This example uses the default location.

*press enter*

```
Set full read/write access for files for all users [Y/n]: 
```
The RCS service will need read/write access to the repository.

*press enter*

```
----------------------------------------------------------------------------
Database Type

The type of Database

[1] MongoDB: MongoDB
[2] DocumentDB: DocumentDB
Please choose an option [1] : 
```
Choose MongoDB.

*press enter*


```
----------------------------------------------------------------------------
MongoDB Database Setup

You can connect to an existing MongoDB installation, or install a new instance on this machine. Note that you will likely have to edit your Firewall settings to allow the Deadline Clients to communicate with the MongoDB database.

[1] Install a new MongoDB database on this machine?
[2] Connect to an existing MongoDB database installation?
Please choose an option [2] : 1
```
Since this is a first time installation choose install a new MongoDB.

*press 1, then enter*

```
MongoDB's open file descriptor limit (ulimit -n) will be set to the recommended value of 200000.

You can change this limit in /etc/init.d/Deadline10db after the installation has finished, but MongoDB will have to be restarted before it recognizes the change. Refer to the Deadline installation documentation for more information on resource limits.

Do you wish to continue? [Y/n]:
```
Accept the default value

*press enter*

```
MongoDB Installation Type

You can have the installer download MongoDB automatically, or you can download the pre-packaged MongoDB binaries yourself and specify the location of the binaries package. Make sure to download the MongoDB binaries package (and not the source package), and note that the ZIP package is for Windows installations and the TAR.GZ package is for Linux and Mac OS X installations.

[1] Download MongoDB (requires internet connection)
[2] Pre-packaged Binaries
Please choose an option [1] : 2 
```
The MongoDB installer was already downloaded in [this section](#downloading-software). Select option 2 to use the already downloaded installer.

*press 2 then enter*

```
Pre-packaged Binaries []: /home/admin/mongodb-linux-x86_64-debian10-4.4.23.tgz
```
Enter the path to where the installer file was downloaded then press enter.

```
----------------------------------------------------------------------------
MongoDB Installation

This is the directory that the MongoDB database will be installed to.

MongoDB Directory [/opt/Thinkbox/DeadlineDatabase10]: 
```
Accept the default value

*press enter*

```
This is the hostname and/or IP address at which client machines will attempt to 
reach this database.

MongoDB Hostname [deadline;10.0.10.15]:
```

> [!Note] the installer may use the loopback IP address `127.0.0.1` as the default option. If this is the case for your installation you need to add the correct IP address or it will throw an error.
> 
> If you don't know the ip you can find it using the below command which list all the IP addresses in use.
> ```bash
> ip address | grep inet
> ```
> If you have multiple network interfaces the command will list the IP addresses for all of them. You will need to choose the one you intend for the clients to connect to.


By Default the installer chooses the loopback IP address which can cause problems. This is the address clients will use to connect to the MongoDB server. Enter the hostname and ip address sepperated by `;` For example `deadline;10.0.10.15`


```
MongoDB Port [27100]: 
```
Accept the default value

*press enter*

```
-The MongoDB Service For Deadline 10 is already running (PID 11638).
----------------------------------------------------------------------------
MongoDB Security

Require client authentication via SSL/TLS [Y/n]: 
```
If this is a production deployment SSL/TLS is highly recommended.

*press enter*

```
----------------------------------------------------------------------------
Make a note of the location chosen for the Certificate Directory. Deadline 
Clients connecting with a 'Direct Connection' will need the 
'Deadline10Client.pfx' file that is generated here.

Certificate Directory [/opt/Thinkbox/DeadlineDatabase10/certs]: 
```
Accept the default value

*press enter*

```
Certificate Password :
```
If using RCS only this is unnecessary as the key will remain secure on the server and not shared with the clients.

If using Direct Connect then its a good idea to password protect the certificate as it will need to be shared with all the clients connecting to the MongoDB database.

```
Use client certificate for DB user authentication [Y/n]: 
```

*press enter*

```
----------------------------------------------------------------------------
Secrets Management Setup

Install Secrets Management

Secrets Management is a feature for securely managing storage and access of secrets for your render farm.
                        NOTE: Secrets Management will only work if SSL/TLS communication is enabled.

Install Secrets Management

[1] Enable Secrets Management (Strongly Recommended)
[2] Skip the Secrets Management installation
Please choose an option [1] : 
```
If planning to use AWS burst rendering in the cloud its a good idea to have Secrets Manager installed to protect the AWS access keys.

```
----------------------------------------------------------------------------
Secrets Management Setup - Create Admin User

Specify the credentials to create the initial admin user. Don't forget the 
credentials as they will later be used to access secrets. 

Admin Username []: 
```
Pick a user name for the key admin

```
Passwords must be at least 8 characters long and contain at least one lowercase 
letter, uppercase letter, digit and a special character.

Password :
```
Create a password for the key admin user

```
Import settings from a previous Repository export [y/N]: 
```
This is a clean install so no need to import settings.

*press enter*

```
----------------------------------------------------------------------------
Setup is now ready to begin installing Deadline Repository on your computer.

Do you want to continue? [Y/n]: 
```
*press enter*

```
----------------------------------------------------------------------------
Please wait while Setup installs Deadline Repository on your computer.

 Installing
 0% ______________ 50% ______________ 100%
 #########################################
```

Deadline Repository should now be installed and running. If you want to use a direct connect you can finish here. If you want to use RCS continue to the next section.


## Install Remote Connection Server (RCS)

The Remote Connection Server acts as a relay between the clients and server. In this example the RCS is installed on the same server as the Deadline Repository, however it can be run on a seperate server.

**preparation**

Create user account for RCS to run as

```bash
sudo useradd -m deadline
```

Before starting the permissions for the deadline certificate need to be changed so the RCS can run as a normal user.

Currently root owns the certificate, so change the owner to the user that the RCS service will run as, which in this case is `deadline`.

```bash
sudo chown deadline: /opt/Thinkbox/DeadlineDatabase10/certs/Deadline10Client.pfx
```

**installation**

Start the installation by running the installer.
```bash
sudo ./DeadlineClient-10.*-linux-x64-installer.run
```


```
----------------------------------------------------------------------------
Welcome to the Deadline Client 10.2.0.10 Setup Wizard.

----------------------------------------------------------------------------
Please read the following License Agreement. You must accept the terms of this 
agreement before continuing with the installation.

Press [Enter] to continue:
By downloading or using this software, you agree to the AWS Customer Agreement 
(https://aws.amazon.com/agreement/) and AWS Intellectual Property License 
(https://aws.amazon.com/legal/aws-ip-license-terms/). You acknowledge that this 
software is AWS Content as defined in those Agreements.

Press [Enter] to continue:

Do you accept this license? [y/n]: y
```
Like with the repository installation, accept the EULA to bein installing.

```
----------------------------------------------------------------------------
Client Installation Directory

Please specify the directory where Deadline Client will be installed.

Installation Directory [/opt/Thinkbox/Deadline10]: 
```
This can be left as default.

*Press enter to continue*

```
Set full read/write access for files for all users [y/N]: 
```
Only root needs to be able to change the files in the install directory, so this can be left as default.

*Press enter to continue*

```
----------------------------------------------------------------------------
Select Installation Type

Please pick what to install

This page allows to choose between installation of various setups each of which require client setup.

[1] Client: Installs all components which are needed for a Deadline client.
[2] Remote Connection Server: Select this option if you are installing a server. For client installation RCS is not needed.
[3] Deadline Web Service: The Deadline Web Service application is a command line application for the Deadline render farm management system. It allows you to get query information from Deadline over an Internet connection.
Please choose an option [1] : 2
```
Select the Remote Connection Server option.

*Press 2 then enter*

```
----------------------------------------------------------------------------
Deadline Setup

The network path to the Deadline Repository.

Repository Directory [/opt/Thinkbox/DeadlineRepository10]: 
```
If the repository was installed in the default directory this option can be left as default. If not enter the path to the repository directory.

*press enter*

```
----------------------------------------------------------------------------
Deadline Setup

The Database TLS certificate (Deadline10Client.pfx). By default, it can be found 
in the Deadline Database installation in the "certs" folder. Leave blank if the 
Database does not require a certificate.
                    
This is not the same as the certificate that is used for the Remote Connection 
Server (Deadline10RemoteClient.pfx). 

Database TLS Certificate [/opt/Thinkbox/DeadlineDatabase10/certs/Deadline10Client.pfx]: 
```
If the certificate is in the default location press enter to continue. If not enter the path to the certificate.

*press enter*

```
The password for the Database TLS certificate. Leave blank if the TLS 
certificate does not require a password.

TLS Certificate Password :
```
If the certificate is password protected enter the password now. If not then press enter to continue.

*press enter*


```
----------------------------------------------------------------------------
Secrets Management Initial Server Setup

It appears that you are installing a Remote Connection Server to a repository 
with Secrets Management enabled.

Would you like to assign this machine a server role and grant it access to the master key? This is highly recommended, otherwise the server role and master key can be granted by running DeadlineCommand after the installation.

[1] Assign server role and grant master key access. (Strongly recommended)
[2] Continue without assigning server role or granting master key access.: You will need to manually assign the Remote Connection Server a server role and master key access after the installation using deadlinecommand.
Please choose an option [1] : 
```
The server role needs to be assigned to the RCS server so it can access secrets.

*press enter*


```
----------------------------------------------------------------------------
OS User Credentials

Please enter the name of the user that will be running the Remote Connection 
Server. Note: This user must have access to the database certificate.

Username [root]: deadline
```
Not recommended to use root. This example is using an already existing user account called `deadline`.

*type deadline then enter*

```
----------------------------------------------------------------------------
Secrets Management Credentials

Enter the credentials of the Admin User. These credentials will be used to 
assign a server role and grant master key access to the this machine.

Admin Username []:
```
Enter the Secrets Manager admin user name created during the repository installation.

```
Passwords must be at least 8 characters long and contain at least one lowercase 
letter, uppercase letter, digit and a special character.

Password :
```
Enter the Secrets Manager admin password created during the repository installation.

```
If you have not changed the master key, leave this field untouched. (The default 
name of the master key is "defaultKey".)

Name of the current master key [defaultKey]: 
```
This can be left as default.

*press enter*

```
----------------------------------------------------------------------------
Deadline Launcher Setup

The Launcher allows for remote communication between Deadline Clients.

Install Launcher As A Daemon [y/N]: 
```

This will create a service that starts the Launcher when the system boots up.

I prefer to create my own service to start RCS and avoid unwanted services from starting with Launcher such as the Worker service.

No need to run the launcher as a Daemon.

*press enter*

```
----------------------------------------------------------------------------
Block Auto Update Override

Auto upgrade is a feature of Deadline that permits Deadline to upgrade itself based on files in the repository. Although useful, we recommend customers block this feature unless they need it as this increases security.

[1] Block auto upgrade via a secure setting. (Recommended): This option disables auto upgrade on this install, ignoring any auto upgrade settings in the repository.
[2] Use the repository settings to determine if auto upgrade is enabled.: This will use the repository (and the local override setting, if any) to determine if auto upgrade is enabled or disabled.
Please choose an option [1] : 2
```
Auto upgrade is a very useful feature when managing a large render farm on-prem. My recommendation is to enable it unless you have another method of rolling out updates to the render nodes.

*press 2 then enter*

```
----------------------------------------------------------------------------
Remote Connection Server (RCS)

User/Group running the RCS [admin]: deadline
```
Enter the user name from the user created earlier.

*press enter*

```
Unsecured port for Deadline Clients [8080]: 
```
Leave as default.

*press enter*

```
Require external Clients to use TLS [Y/n]: 
```
Highly recommended if running in a production environment to enable. If in a testing environment its best to leave off to make troubleshooting easier.

*press enter*

```
----------------------------------------------------------------------------
If TLS is enabled, only local connections will be allowed to communicate over 
the unsecured HTTP port. External connections will be served over the TLS Port.

NOTE: If Secrets Management is enabled in the repository installer, TLS MUST be 
enabled or AWS Portal will NOT work.

Secured Port [4433]: 
```
Leave as default.

*press enter*

```
Launcher starts RCS whenever RCS isn't running [Y/n]: 
```
Leave as default.

*press enter*

```
----------------------------------------------------------------------------
HTTPS Server Settings

TLS requires the use of X509 Certificates to validate the authenticity of both the client and server. You can either provide your own, or have the installer generate a new set for you.

TLS (HTTPS) Certificates

[1] : Generate New Certificates
[2] : Use Existing Certificates
Please choose an option [1] : 
```
As this is a new install it need to generate a new certificate for the RCS to secure communication between RCS and the workers.

*press enter*

```
----------------------------------------------------------------------------
Generate New Certificates

----------------------------------------------------------------------------
HTTPS Server Settings

Make a note of the location chosen for the Certificate Directory. Deadline 
Clients connecting to a 'Remote Connection Server' will need the 
'Deadline10RemoteClient.pfx' file that is generated here.

Certificate Directory [/opt/Thinkbox/Deadline10/certs]: 
```
Leave as default so the certificate is put with the other certificates.

*press enter*

```
Setting a password on the client certificate will encrypt the private key 
information, and require a password in conjunction with the certificate at 
connection time (recommended).

Certificate Password :
```
Not necessary unless sharing the certificate with clients or moving it off the server.

*press enter*

```
----------------------------------------------------------------------------
Setup is now ready to begin installing Deadline Client on your computer.

Do you want to continue? [Y/n]: 
```
*press enter*

```
----------------------------------------------------------------------------
Please wait while Setup installs Deadline Client on your computer.

 Installing
 0% ______________ 50% ______________ 100%
 ########################################
```

Congratulations! The Deadline Repository is now set up and ready for Workers to connect to it and render.

### Create systemd service to auto launch RCS on startup
To ensure Deadline RCS starts when the server is rebooted we need to create a systemd service. 

*Note: If you selected run Launcher as daemon this step is unnecessary.*

Create service file in `/etc/systemd/system/`

Below is a template, if you did not use the same user/install path as this instruction make sure to update `User` `Group` and `ExecStart` with the correct details.

You will need to change the `User` and `Group` values to the name of the uesr that runs Deadline RCS.

> [!Note] you need root permissions, `sudo su`
```bash
cat << EOF >> /etc/systemd/system/deadlinercs.service
[Unit]
Description=Deadline Remote Connection Server
After=network.target

[Service]
Type=simple
User=deadline
Group=deadline
ExecStart=/opt/Thinkbox/Deadline10/bin/deadlinercs
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
EOF
```

Enable and start the service
```bash
systemctl enable --now deadlinercs.service
```

Check to see if the service started successfully
```bash
systemctl status deadlinercs.service
```

---

### What next?

Now that you have a your Deadline Repository set up, you need some render nodes to connect to it. 

[Follow this guide here to continue building your own render farm.](../deadline-linux-worker)

---

### Troubleshooting

If this error appears after installation it means that the user running the RCS server was unable to read the repository certificate. Change the owner of the repository certificate to the user running the RCS server and reinstall RCS.

```
Warning: Problem running post-install step. Installation may not complete 
correctly
 Secrets Management: Failed to assign a Server role and grant key access to this 
machine. You will need to run 'deadlinecommand Secrets ConfigureServerMachine' 
after the installation. Check the bitrock_installer log file in /tmp for more 
information.
Press [Enter] to continue:
```

---


#### Logs and errors

Deadline logs are kept in `/var/log/Thinkbox/Deadline10`

By running `tail -f` on the log file for the RCS server and trying to connect the Worker to the Repository I was able to find a useful clue as to what is going on.
```bash
tail -f /var/log/Thinkbox/Deadline10/deadlinercs-deadline-2023-03-07-0000.log
```

Here is the error I found in the log, which indicates the RCS server is unable to connect to the MongoDB.
```
2023-03-07 01:15:48:  Error: Could not connect to any of the specified Mongo DB servers defined in the "Hostname" parameter of the "settings/connection.ini" file in the root of the Repository.
```
