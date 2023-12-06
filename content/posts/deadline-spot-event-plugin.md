+++
title = "Deadline Spot Event Plugin How To Guide"
date = "2023-08-15T13:20:43+10:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["DEADLINE", "AWS", "RENDER FARM"]
keywords = ["thinkbox", "deadline", "aws", "spot", "render farm"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
Toc = true
color = "" #color from the theme settings
+++

<!-- 
NOTES:

https://medium.com/trackit/deploying-aws-thinkbox-deadline-using-the-render-farm-deployment-kit-rfdk-6ffa4bb4626d

https://docs.thinkboxsoftware.com/products/deadline/10.3/1_User%20Manual/manual/event-spot.html
 -->

The Spot Event Plugin for Deadline allows Deadline to spin up fleets of render workers on AWS using EC2 Spot.

## Prerequisites

You'll need an AWS Account, and an AMI with Deadline Worker and rendering software installed. [Here](/posts/deadline-linux-worker) is a guide to installing Deadline Client on Linux if you don't have one already.

## AWS configuration

### Create IAM User

This section covers giving Deadline access and permissions it needs to manage resources in AWS.

Create new IAM user
```bash
aws iam create-user --user-name DeadlineSpotEventPluginAdmin
```

Create access key

*Access keys are how Deadline authenticates to AWS. Handle them with care as if they were passwords and don't publish or share them.*

```bash
aws iam create-access-key --user-name DeadlineSpotEventPluginAdmin
```

Save the `AccessKeyId` and `SecretAccessKey` somewhere safe like [keepass](https://keepassxc.org/), we'll need them for later.

Attach policies

*Policies say what IAM users can and can't do in AWS*

AWSThinkboxDeadlineSpotEventPluginAdminPolicy and AWSThinkboxDeadlineResourceTrackerAdminPolicy

```bash
aws iam attach-user-policy \
  --policy-arn arn:aws:iam::aws:policy/AWSThinkboxDeadlineSpotEventPluginAdminPolicy \
  --user-name DeadlineSpotEventPluginAdmin
```

```bash
aws iam attach-user-policy \
  --policy-arn arn:aws:iam::aws:policy/AWSThinkboxDeadlineResourceTrackerAdminPolicy \
  --user-name DeadlineSpotEventPluginAdmin
```

There are 2 extra policies we need to add that are not in the documentation, IAMReadOnlyAccess and AmazonEC2ReadOnlyAccess. Without them you will get an error when launching the Spot Event Configuration Utility.

```bash
aws iam attach-user-policy \
  --policy-arn arn:aws:iam::aws:policy/IAMFullAccess \
  --user-name DeadlineSpotEventPluginAdmin
```

```bash
aws iam attach-user-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess \
  --user-name DeadlineSpotEventPluginAdmin
```


### Create IAM EC2 Role

Create spot worker IAM role.

First create a new policy document that allows the role to be assigned to EC2.

```bash
cat <<EOF > ec2-assume-role-policy.json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Principal": {"Service": "ec2.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }
}
EOF
```

Create a new role that will give the EC2 render workers the necessary permissions.

```bash
aws iam create-role \
  --role-name DeadlineSpotWorker \
  --assume-role-policy-document file://ec2-assume-role-policy.json
```

Create an instance profile so the role can be attached to an EC2 instance.

```bash
aws iam create-instance-profile \
  --instance-profile-name DeadlineSpotWorker

aws iam add-role-to-instance-profile \
  --instance-profile-name DeadlineSpotWorker \
  --role-name DeadlineSpotWorker
```

Attach the spot worker policy to the new IAM role.

```bash
aws iam attach-role-policy \
  --role-name DeadlineSpotWorker \
  --policy-arn arn:aws:iam::aws:policy/AWSThinkboxDeadlineSpotEventPluginWorkerPolicy 
```


### Create Resource Tracker IAM Role

Start with creating the trust policy file.

```bash
cat <<EOF > lambda-assume-role-policy.json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Principal": {"Service": "lambda.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }
}
EOF
```

Create a new IAM role.

```bash
aws iam create-role \
  --role-name DeadlineResourceTrackerAccessRole \
  --assume-role-policy-document file://lambda-assume-role-policy.json
```

Add policies to the IAM role.

```bash
aws iam attach-role-policy \
  --role-name DeadlineSpotWorker \
  --policy-arn arn:aws:iam::aws:policy/AWSThinkboxDeadlineResourceTrackerAccessPolicy
```

### Spot Fleet Tagging IAM Role

This is supposed to be automatically created by Deadline, but I don't personally like the idea of giving Deadline the ability to create any IAM role it wants. So let's create it manually instead.

Like before start with creating the trust policy file.

```bash
cat <<EOF > spotfleet-assume-role-policy.json
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Principal": {"Service": "spotfleet.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }
}
EOF
```

Create the new IAM role.

```bash
aws iam create-role \
  --role-name aws-ec2-spot-fleet-tagging-role \
  --assume-role-policy-document file://spotfleet-assume-role-policy.json
```

Attach policies to the IAM role.

```bash
aws iam attach-role-policy \
  --role-name aws-ec2-spot-fleet-tagging-role \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEC2SpotFleetTaggingRole
```

## Deadline Configuration

### Enabling Spot Event Plug-in

Open the Deadline Monitor

Enable Superuser by clicking in the top menu `Tools -> Super User Mode`

Then open Tools -> Configure Events...

From the list on the left hand side select Spot.

1. Set 'State' to 'Global Enabled' and 'Enable Resource Tracker' to 'True'.
2. Put your AWS Access Key and Secret here.
3. Set the region to where you want rendering to happen, ideally a region geographically close to you.

![deadline event menu spot](/posts/deadline-spot-event-plugin/deadline-event-menu-spot.png)

### Configuring Spot Fleets

In the Deadline Monitor top menu bar go to `Scripts -> Configuration -> Spot Event Configuration Utility`.

Click on 'Quick Create'

![deadline spot event config window](/posts/deadline-spot-event-plugin/deadline-spot-event-config-window.png)

<!-- https://docs.thinkboxsoftware.com/products/deadline/10.3/1_User%20Manual/manual/event-spot.html -->

This will open the 'Quick Create' window where you can configure your spot fleet.

1. Give the spot fleet a name to make it easy to identify.
2. Select the group. When a job is submitted to this group it will trigger the spot event fleet creation.
3. Select IAM to add the worker to.
4. Select instance types.
5. (Optional) Choose a key pair so you can access the works via SSH if needed.
6. Select pools to add the worker to.

[Learn more about how Pools and Groups work here.](https://docs.thinkboxsoftware.com/products/deadline/10.3/1_User%20Manual/manual/pools-and-groups.html)

![deadline spot fleet quick create 1](/posts/deadline-spot-event-plugin/deadline-spot-fleet-quick-create-1.png)

1. Switch to 'Network Settings' tab.
2. Select the VPC to launch the render workers in.
3. Select subnets to launch the render workers in. Highly recommend launching in private subnets only and selecting at least 1 subnet from each availability zone to spread out the render workers to reduce interruptions.
4. Select security group that allows the workers network access to any resources they need.
5. Set the instance profile to 'DeadlineSpotWorker' we made [here](#create-iam-ec2-role).

![deadline spot fleet quick create 2](/posts/deadline-spot-event-plugin/deadline-spot-fleet-quick-create-2.png)

1. Switch to 'Misc. Settings' tab.
2. (Optional) Add a start up script here.
3. Add tags. Highly recommend setting at least a 'Name' tag to make the render workers easy to identify in the AWS console.

![deadline spot fleet quick create 3](/posts/deadline-spot-event-plugin/deadline-spot-fleet-quick-create-3.png)

1. Switch to 'Spot Fleet Settings' tab.
2. Set how many render workers you want to launch in the spot fleet.
3. Make sure it is set to 'Terminate' any instances when complete or interrupted.
4. (Optional) Tick 'Replace unhealthy instances' so new instances are started if any are interrupted.
5. Select 'Disable' auto assign public IP as render workers should never have direct internet access.
6. Click the 'Create' button to create a new spot fleet.

![deadline spot fleet quick create 4](/posts/deadline-spot-event-plugin/deadline-spot-fleet-quick-create-4.png)


## Troubleshooting

If you get this error when opening the 'Spot Event Configuration Utility' it means Deadline cannot create the IAM role it needs to manage the spot fleet. [Here is the guide to manually add the role](#spot-fleet-tagging-iam-role).

![deadline spot script config error](/posts/deadline-spot-event-plugin/deadline-spot-fleet-script-config-error-message.png)