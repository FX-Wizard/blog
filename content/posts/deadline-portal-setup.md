+++
title = "Deadline Portal on Linux Setup"
date = "2023-08-09T21:02:26+10:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["deadline", "linux", "how to", "render farm"]
keywords = ["deadline", "aws", "portal", "render farm", "thinkbox"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

How to set up Deadline AWS Portal on a dedicated Linux server.

AWS Portal has quite a few moving parts, so to keep things simple this guide will focus on the following.

- Portal installation
- basic configuration
- creating a worker fleet.

If you want to learn more about the Portal and hybrid cloud rendering [Here is the link to the offical documentation](https://docs.thinkboxsoftware.com/products/deadline/10.3/1_User%20Manual/manual/aws-portal-installing.html)

**Prerequisites**

This guide assumes you already have the following
- Deadline Repository with RCS. [Here is the guide to set one up](/posts/deadline-linux-server)
- Linux server to install the Portal
- AWS account

---

### Using the AWS CLI

The easiest way to get up and running with the AWS CLI is to use the Cloud Shell in the AWS Console.

There are other ways of setting up the AWS CLI which [you can find here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)

I am using v2 of the AWS CLI in this guide.

## IAM User

The Portal needs to be able to access AWS resources on your behalf to automate the deployment and management of the cloud render farm.

**Admin user**

Admin user account is for Portal to do the initial set up of all the things it needs in AWS. It can be deleted after the set up is complete.

Create new IAM user
```bash
aws iam create-user --user-name AWSPortalAdmin
```

Create access key

*Access keys are how Deadline authenticates to AWS. Handle them with care as if they were passwords and don't publish or share them.*

```bash
aws iam create-access-key --user-name AWSPortalAdmin
```

Save the `AccessKeyId` and `SecretAccessKey` somewhere safe like [keepass](https://keepassxc.org/), we'll need them for later.

Attach policies

*Policies are what control what IAM users can and can't do in AWS*

```bash
aws iam attach-user-policy \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess \
  --user-name AWSPortalAdmin
```

**Portal user**

Portal user account is for the management of fleets and other resources Deadline will use when running render jobs.

Create new IAM user
```bash
aws iam create-user --user-name AWSPortal
```

Create access key
```bash
aws iam create-access-key --user-name AWSPortal
```
Like with the first access key make sure to save this one somewhere safe.

We need to attach 2 policies, AWSThinkboxAWSPortalAdminPolicy, and AWSThinkboxDeadlineResourceTrackerAdminPolicy, to the newly created IAM user. 

```bash
aws iam attach-user-policy \
  --policy-arn arn:aws:iam::aws:policy/AWSThinkboxAWSPortalAdminPolicy \
  --user-name AWSPortal
```

```bash
aws iam attach-user-policy \
  --policy-arn arn:aws:iam::aws:policy/AWSThinkboxDeadlineResourceTrackerAdminPolicy \
  --user-name AWSPortal
```

> [!Hint]
> search for IAM policies using CLI
> ```bash
> aws iam list-policies | grep thinkbox -i
> ```

---

## Install AWS Portal Server


### Prerequisites

Ideally installed on its own virtual server since it needs to be always on (or at least on when any rendering is going to happen).

Needs access to Deadline Repository and file server where assets are stored

Needs Deadline Client installed on the same server. [Link to instructions here](/posts/deadline-linux-worker)


### Download and extract

```bash
curl -O https://thinkbox-installers.s3.us-west-2.amazonaws.com/Releases/Deadline/10.3/1_10.3.0.9/Deadline-10.3.0.9-linux-installers.tar
```

Run installer
```bash
sudo -E ./AWSPortalLink-1.3.0.3-linux-x64-installer.run
```

> **Note**
>
> If you get an error like this.
> 
> ```
> Error: There has been an error.
> Missing DEADLINE_PATH environment variable.  Please ensure that Deadline is 
> installed.  If you just installed Deadline from this console, then please try 
> running this installer in a new console. If installing with sudo, add '-E' 
> option to sudo.
> Press [Enter] to continue:
> ```
> 
> The environment variable that points to the Deadline Client is missing. This should have been set by the Deadline client installer, if you have only just installed the client either reboot or set the environment variable manually to point to the `bin` folder in Deadline's installation directory, then try installing again.
> 
> ```bash
> export DEADLINE_PATH=/opt/Thinkbox/Deadline10/bin
> ```

```
----------------------------------------------------------------------------
Welcome to the AWSPortalLink Setup Wizard.

----------------------------------------------------------------------------
Please read the following License Agreement. You must accept the terms of this 
agreement before continuing with the installation.

Press [Enter] to continue:


Do you accept this license? [y/n]: 
```

Accept the EULA to continue.

```
----------------------------------------------------------------------------
Select the components you want to install; clear the components you do not want 
to install. Click Next when you are ready to continue.

AWSPortalLink [Y/n] :

AWSPortalAssetServer [Y/n] :

Is the selection above correct? [Y/n]: 
```

We want to install both AWSPortalLink and AWSPortalAssetServer together.

*press enter x3 times*

```
----------------------------------------------------------------------------
The directory where AWSPortalLink will be installed.

Installation Directory [/opt/Thinkbox/AWSPortalLink]:     
```

*press enter*

```
----------------------------------------------------------------------------
Portal Link Public IP Override (Optional)

Override the IP address that AWS Portal accepts traffic from. If left blank, we 
will use the automatically detected IP: '13.55.198.10' .

Portal Link Public IP Override (Optional) []: 
```

```
----------------------------------------------------------------------------
The directory where AWSPortalAssetServer will be installed.

Installation Directory [/opt/Thinkbox/AWSPortalAssetServer]: 
```

*press enter*

```
----------------------------------------------------------------------------
AWS Identity and Access Management (IAM) Resource Creation

AWS Portal requires IAM resources in your AWS account to operate.
Create these IAM resources:

[1] Automatically now: Requires your Administrator AWS credentials.
[2] Manually later: Run a resource creation script after the installer completes, or create the resources manually.
Please choose an option [2] : 
```

Since we have already set up the admin IAM user we can choose option 1 to set up the resources in AWS now.

*press 1 then enter*

```
Learn more [Y/n]:
```

This will give additional explanation of what resources will be created in your AWS account.

*press n then enter*

```
See the IAM resource creation script [Y/n]: 
```

This will print the entire resource creation script to the terminal. This is purely for transparency and not needed for installation.

*press n then enter*

```
----------------------------------------------------------------------------
AWS Credentials

Please enter your Administrator AWS credentials.

The installer will use these to create IAM roles and users for AWS Portal and 
the Asset Server in your account.

AWS Access Key ID []: 

AWS Secret Access Key :
```

Enter the AWSPortalAdmin key saved earlier.

```
----------------------------------------------------------------------------
AWS Portal Asset Server Root Directories

Please enter the roots of all directory structures (eg. the root of a file 
server) the asset server should have access to. Files in directory structures 
not listed here cannot be used to render with AWS Portal. If you need more than 
3 directory structures, or need to change them later, please use the Asset 
Server Settings dialog in the Deadline Monitor (Tools > Asset Server Settings)

Directory 1 []: 
```

Deadline needs to copy things like 3D models, textures, and other files needed by the renderer to the cloud. To do that Portal needs to know where those files are kept and be able to access them.

Enter the paths to any shared drives or file servers where you keep the assets for your projects.

This can be changed later in the Deadline Monitor. So if you are unsure or want to do this later just press enter to continue.

```
----------------------------------------------------------------------------
Setup is now ready to begin installing AWSPortalLink on your computer.

Do you want to continue? [Y/n]:
```

*press enter*

---

### Troubleshooting

```
Running the AWS Portal Link service requires an open file descriptor limit (ulimit -n) to be increased. This value limits the number of active network connections that AWS Portal Link can have at a time. We have increased it to our recommended value of 65535, from the default of 1024.

You can change this limit in

/etc/init.d/awsportallinkservice

after the installation has finished, but the AWS Portal Link service will have to be restarted before it recognizes the change. Refer to the Deadline installation documentation for more information on resource limits.

Do you wish to continue? [Y/n]: 
```

If you get this error you'll need to restart the Portal services after the installation. Or just reboot the system.
```bash
systemctl restart awsportallink
systemctl restart awsportalassetserver
systemctl restart awsportallinkservice.service
systemctl restart awsportalassetservershellscript.service
```

---

### Clean up

Don't forget to expire the admin user's access key since we don't need it anymore

```bash
aws iam delete-access-key --access-key-id <access key id> --user-name AWSPortalAdmin
```

---

## Configure Deadline AWS Portal

To configure the AWS Portal, start by opening the Deadline Monitor.

Tools -> Super User Mode

![enable super user mode](/posts/deadline-portal-setup/deadline-superuser-mode.png)

View -> New Panel -> AWS Portal

![open aws portal panel](/posts/deadline-portal-setup/deadline-monitor-aws-portal.png)

Portal will ask for the AWS access key for the AWSPortal IAM user [created in this section.](/posts/deadline-portal-setup/#iam-user)

Enter the access key and secret key to continue.

![aws portal login](/posts/deadline-portal-setup/deadline-portal-login.png)

The AWS Portal panel will appear. Open the AWS Portal Settings.

![aws portal login](/posts/deadline-portal-setup/deadline-monitor-portal-settings.png)

Set the Connection Server to your newly created Portal server. Then click OK.

![aws portal login](/posts/deadline-portal-setup/deadline-monitor-portal-settings-2.png)

Next we need to configure the asset server settings.

Tools -> Configure Asset Server...

![aws portal login](/posts/deadline-portal-setup/deadline-monitor-configure-asset-server.png)

The settings should have been auto populated, check to see if the IP address matches the one for your Portal server.

![aws portal login](/posts/deadline-portal-setup/deadline-monitor-asset-server-settings.png)


## Continuing set up of AWS Portal

Here is where this guide finishes, you now have a working Portal server ready to be configured for hybrid cloud rendering.

There is much more to be configured and many more settings you can play with. To continue configuration [see the official guides here](https://docs.thinkboxsoftware.com/products/deadline/10.3/1_User%20Manual/manual/aws-portal-configuration-reference.html).
