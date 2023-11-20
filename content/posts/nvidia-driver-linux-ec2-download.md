+++
title = "Downloading AWS Nvidia Driver for EC2 from S3"
date = "2023-11-20T10:31:51+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["AWS", "NVIDIA", "LINUX", "EC2"]
keywords = ["aws", "nvidia", "driver", "ec2", "linux"]
description = "How to download the Nvidia drivers for AWS EC2 instance using the AWS CLI to directly download from the S3 bucket to EC2 instance."
showFullContent = false
readingTime = false
hideComments = false
toc = true
color = "orange" #color from the theme settings
+++

Downloading the official Nvidia driver for AWS EC2 instances can be tricky. Here are my notes on the process of using the AWS CLI to download the driver directly to the EC2 instance you want to install it on.

This guide uses the AWS CLI to create the policy. [AWS Cloud Shell](https://docs.aws.amazon.com/cloudshell/latest/userguide/welcome.html) provides an easy way to use the CLI.

Here are links to official docs from AWS for installing the Nvidia drivers for [Windows](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/install-nvidia-driver.html) and [Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-nvidia-driver.html) for reference.

## Creating IAM policy and Role

The AWS specific Nvidia driver lives in an S3 bucket. We need to give the EC2 instance access to that bucket to download the driver by assigning it an IAM role.

### Create IAM Access Policy

First we need a policy that lets the EC2 instance read the S3 bucket the Nvidia driver is stored in.

Create a file with the Nvidia driver S3 bucket read policy.

```json
cat <<EOF> nvidia-s3-allow-read.json 
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "NvidiaS3AllowRead",
            "Effect": "Allow",
            "Action": [
                "s3:Get*",
                "s3:List*",
                "s3:Describe*"
            ],
            "Resource": [
                "arn:aws:s3:::ec2-windows-nvidia-drivers*",
                "arn:aws:s3:::ec2-linux-nvidia-drivers*",
                "arn:aws:s3:::nvidia-gaming*"
            ]
        }
    ]
}
EOF
```

Create the IAM policy and add the policy file to it

```bash
aws iam create-policy \
  --policy-name nvidia-s3-allow-read \
  --policy-document file://nvidia-s3-allow-read.json
  --description.json "Allows reading from AWS Nvidia driver s3 bucket"
```

**IMPORTANT:** Make sure to take note of the "Arn" that is printed out after the policy has been created, we will need this later.

###  Create IAM Role

Now we have a policy that gives access to the S3 bucket with the Nvidia driver we need a way to apply it to our EC2 instance.

First create a policy file to tell the roll it can only be attached to EC2.

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

Create an IAM role to give the EC2 instance we want to download the driver on to.

```bash
aws iam create-role \
  --role-name nvidia-s3-allow-ec2-read \
  --assume-role-policy-document file://ec2-assume-role-policy.json
```

Attach the policy we created just before that gives read access to the Nvidia driver bucket.

In the `--policy-arn` section add the "Arn" you noted down earlier in when creating the policy.

```bash
aws iam attach-role-policy \
  --role-name nvidia-s3-allow-ec2-read \
  --policy-arn arn:aws:iam::<account ID>:policy/nvidia-s3-allow-read
```

Create instance profile that will attach the new role to the EC2 instance.

```bash
aws iam create-instance-profile \
  --instance-profile-name nvidia-s3-allow-ec2-read-instance-profile
aws iam add-role-to-instance-profile \
  --role-name nvidia-s3-allow-ec2-read \
  --instance-profile-name nvidia-s3-allow-ec2-read-instance-profile
```

### Attatch IAM role to EC2 instance

Attach the newly created IAM role to the EC2 instance. This can be done either through the console or via the CLI.

Here is the CLI method. You will need the instance ID of the EC2 instance you want to assign the role to.

```bash
aws ec2 associate-iam-instance-profile \
  --instance-id <EC2 instance InstanceId> \
  --iam-instance-profile Name="nvidia-s3-allow-ec2-read-instance-profile"
```

*Note: EC2 instances can only have 1 IAM role at a time, if your instance already has an IAM role you will need to remove it before applying a new one.*

## Downloading the Nvidia Driver with AWS CLI

Remote into the EC2 instance with SSH

Install AWS CLI

**Ubuntu**
```bash
sudo apt update
sudo apt install -y awscli
```

**Other Linux**
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Download driver using the AWS CLI

```bash
aws s3 cp --recursive s3://ec2-linux-nvidia-drivers/latest/ .
```

---

With the Nvidia drivers downloaded all thats left to do is install them. To help get you started, here are some guides for setting up a 3D workstation on [Ubuntu](/posts/ubuntu-22.04-3d-ws-aws) or [Rocky 9](/posts/rocky-9-3d-ws-aws).