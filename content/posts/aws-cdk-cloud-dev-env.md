+++
title = "AWS CDK - Cloud Development Environment"
date = "2023-11-13T20:14:11+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["AWS", "CDK"]
keywords = ["aws", "cdk", "vscode"]
description = "How to create a development environment in the cloud for writing AWS CDK"
showFullContent = false
readingTime = false
hideComments = false
toc = true
color = "orange" #color from the theme settings
+++

<!-- TODO: give an explanation of the basic architecture -->
**What we are going to build**

We are going to deploy a development environment in the cloud for AWS CDK (Cloud Deployment Kit).

The EC2 instance will be deployed in the default [VPC](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html) (Virtual Private Cloud).

**Why not AWS Cloud9?**

This is more my personal option, I find it easier to work in VS Code for writing CDK.

Some advantages to VS Code over Cloud9.
- Vast ecosystem of extensions.
- Fast IntelliSense, which really speeds up CDK development.
- Useful for more than just AWS cloud development.

One big advantage to Cloud9 is it works out of the box. If you want to use Cloud9 you need not read any further since it has almost everything you need for CDK already. However if you are like me and find it clunky or just perfer working with tools your already familiar with then please read on.

## Getting started

[VS Code](https://code.visualstudio.com/)

Because this guide uses VS Code as the code editor you will need a copy of it installed on your local computer.

We will be using the `Remote - SSH` extension to connect the local VS Code to an EC2 instance running in the cloud where all the processing will happen.

## Spinning up an EC2 instance

If you already have an EC2 instance spun up and ready you can skip to the [next step here.](#create-admin-iam-role)

**Finding the image ID**

Fist we need to find an image to launch the EC2 instance from. I prefer using Ubuntu due to it being easy to set up, but if you're feeling adventurous you can try a Linux distro of your choosing.

Make sure to change `--region=` to the region you want to launch in.

```bash
aws ec2 describe-images --output json --region="us-west-1" \
--owners amazon \
--filters "Name=name,Values=ubuntu*22.04*server*" "Name=architecture,Values=x86_64" \
--query "sort_by(Images, &CreationDate)[-1].{Name: Name, ImageId: ImageId}"
```

Make a note of the `ImageID`, we will need this later

*Breakdown of the `describe-images` command*
* \-\-owners = who owns the image, for example 'self' for your images or 'amazon' for official images
* \-\-filters = getting only the info we are looking for
    * name - filters based on the image name and uses wildcards to fill in the unknown.
        * ubuntu = name of OS
        * 22.04 = release version of the OS
        * server = find only the server version and not the minimal or pro versions.
    * architecture - make sure its x86 for Intel/AMD CPUs. If you want to use ARM based Graviton instances change this to arm64
* \-\-query = pulling the useful info out of the results
    * sort_by(Images, &CreationDate)[-1] = get the latest verson
    * {Name: Name, ImageId: ImageId} = return only the Name and ImageID

If you want to learn more here is a useful article from the makers of Ubuntu going into more detail about finding the right image via CLI

https://ubuntu.com/tutorials/search-and-launch-ubuntu-22-04-in-aws-using-cli#1-overview

**Create new SSH keys**

To securely access the EC2 instance we need to create an SSH key.

*Note: If you are using powershell change the `--output text` value to `| out-file -encoding ascii -filepath cdkDevEnvKey.pem`*.

```bash
aws ec2 create-key-pair \
  --key-name "cdkDevEnvKey" \
  --key-type ed25519 \
  --output text > cdkDevEnvKey.pem
```

This will create a new Private key called cdkDevEnvKey.pem that will be saved in the current directory.

Treat this key like you would any other password or secret and store it somewhere safe. 

If you are on Linux or Mac OS change the permissions of the key so only your user account can access it.

```bash
chmod 400 cdkDevEnvKey.pem
```

It's also a good idea to put it in a secure folder such as the `.ssh` folder in your home drive.

```bash
mv cdkDevEnvKey.pem ~/.ssh/
```

If the folder does not exist you can create it

```bash
mkdir ~/.ssh/
chmod 700 ~/.ssh
```

**Create security group**

We need to create a security group that lets you SSH in from your local computer to the EC2 instance.

First we need your public IP address. An easy way to do that is by going to this website https://ipinfo.io/ip


You can also get the IP from the command line using curl.

```bash
curl ipinfo.io/ip
```

Now to create the security group itself using the AWS CLI.

*Note: Make sure to write down the "GroupId" from the output of the command.*

```bash
aws ec2 create-security-group \
  --group-name sshFromMyIP \
  --description "Allow SSH access from my IP"
```

Add rules to the security group

```bash
aws ec2 authorize-security-group-ingress \
  --group-id <"GroupId" of the security group we just made> \
  --protocol tcp \
  --port 22 \
  --cidr <your IP>/32
```

**Spin up a new EC2 instance**

Put the `ImageId` from the previous command as the value for `--image-id`.

*Make sure to write down the "InstanceId" from the command output.*

```bash
aws ec2 run-instances \
  --image-id "<put the ImageID from the describe-images command here>" \
  --count 1 \
  --key-name cdkDevEnvKey \
  --security-groups "sshFromMyIP" \
  --instance-type t3a.medium
```

*If you want to use Graviton change instance-type to t4g.medium*

### Create admin IAM role

The CDK dev enviroment needs access to deploy infrastructure into the AWS account. Typically this is done by creating a user account with programatic access, however since we are running an EC2 instance in the account we want to deploy into we can simply give it an IAM role that grants it access to deploy infrastructure and avoid having to handle secret access keys.

Create a policy file to tell the role only EC2 can use it.

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

Create the IAM role and an instance profile so the role can be assigned to an EC2 instance.

```bash
aws iam create-role \
  --role-name CDK-EC2-ADMIN \
  --assume-role-policy-document file://ec2-assume-role-policy.json

aws iam create-instance-profile \
  --instance-profile-name CDK-EC2-ADMIN-PROFILE

aws iam add-role-to-instance-profile \
  --instance-profile-name CDK-EC2-ADMIN-PROFILE \
  --role-name CDK-EC2-ADMIN
```

Give it admin access.

```bash
aws iam attach-role-policy \
  --role-name CDK-EC2-ADMIN \
  --policy-arn arn:aws:iam::aws:policy/AWSThinkboxDeadlineSpotEventPluginWorkerPolicy 
```

Assign the new IAM role to our new EC2 instance.

```bash
aws ec2 associate-iam-instance-profile \
  --iam-instance-profile Name=CDK-EC2-ADMIN-PROFILE \
  --instance-id <the ID of the instance you created>
```

The set up of the EC2 instance is now finished, once in the instance spun up you can SSH in.

## Configuring EC2 instance

Start with updating the system

```bash
sudo apt update
sudo apt upgrade -y
```

CDK requires Node.js so lets install it from [nodesource](https://github.com/nodesource/distributions#ubuntu-versions).
```bash
# Download and import the Nodesource GPG key
sudo apt-get install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

# Set version to install
NODE_MAJOR=18

# Create deb repository
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

# Install node.js
sudo apt-get update
sudo apt-get install nodejs -y
```

Install the AWS CLI

```bash
sudo apt install awscli
```

Install Python's pip package manager and Python virtual environments

```bash
sudo apt install python3-pip
sudo apt install python3-venv
```

We need to set some environment variables so the dev environment knows the default location to deploy AWS resources.
```bash
export CDK_DEFAULT_ACCOUNT=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | sed -nE 's/.*"accountId"\s*:\s*"(.*)".*/\1/p')
export CDK_DEFAULT_REGION=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | sed -nE 's/.*"region"\s*:\s*"(.*)".*/\1/p')
export INSTANCE_ID=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | sed -nE 's/.*"instanceId"\s*:\s*"(.*)".*/\1/p')
export CDK_DEFAULT_VPC=$(aws ec2 describe-instances --instance-id $INSTANCE_ID --query 'Reservations[0].Instances[0].NetworkInterfaces[0].VpcId' | tr -d '"')
echo "# CDK environment variables" >> $HOME/.bashrc
echo "export CDK_DEFAULT_ACCOUNT=$CDK_DEFAULT_ACCOUNT" >> $HOME/.bashrc
echo "export CDK_DEFAULT_REGION=$CDK_DEFAULT_REGION" >> $HOME/.bashrc
echo "export CDK_DEFAULT_VPC=$CDK_DEFAULT_VPC" >> $HOME/.bashrc
```

Install CDK itself

```bash
sudo npm install -g aws-cdk
```

Here is what the script looks like combined together.

```bash
# Update the system
sudo apt update
sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y

# node.js 
sudo apt-get install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
NODE_MAJOR=18
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
sudo apt-get update
sudo apt-get install nodejs -y

# AWS CLI
sudo apt install awscli

# Python
sudo apt install python3-pip
sudo apt install python3-venv

# set account ID env var
export CDK_DEFAULT_ACCOUNT=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | sed -nE 's/.*"accountId"\s*:\s*"(.*)".*/\1/p')
export CDK_DEFAULT_REGION=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | sed -nE 's/.*"region"\s*:\s*"(.*)".*/\1/p')
export INSTANCE_ID=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | sed -nE 's/.*"instanceId"\s*:\s*"(.*)".*/\1/p')
export CDK_DEFAULT_VPC=$(aws ec2 describe-instances --instance-id $INSTANCE_ID --query 'Reservations[0].Instances[0].NetworkInterfaces[0].VpcId' | tr -d '"')

echo "# CDK environment variables" >> $HOME/.bashrc
echo "export CDK_DEFAULT_ACCOUNT=$CDK_DEFAULT_ACCOUNT" >> $HOME/.bashrc
echo "export CDK_DEFAULT_REGION=$CDK_DEFAULT_REGION" >> $HOME/.bashrc
echo "export CDK_DEFAULT_VPC=$CDK_DEFAULT_VPC" >> $HOME/.bashrc

# Setup aws-cdk
sudo npm install -g aws-cdk
```

Now that you know what all the parts of the script do you could use it as a user data script when launching the EC2 instance so it automatically sets up the enviornment.

---

## VS Code set up

To access the CDK development EC2 instance we need to install and configure the SSH extension in VS Code.

Start with opening VS Code.

1. In the side tool bar, click on the extensions icon.
2. Search for 'ssh'.
3. Install 'Remote - SSH' extension by Microsoft.

![vs code install ssh extension](/posts/aws-cdk-cloud-dev-env/vscode-install-ssh-extension.png)

Once installed you'll see a new icon on the side bar.

There are 2 options to add a new ssh connection.

1 - Click on the icon, then hidden in the 'SSH' drop down is a '+' button to add a new host.
 
![vs code install ssh extension](/posts/aws-cdk-cloud-dev-env/vscode-ssh-add-connection-sidebar.png)

The command pallet will open and ask for the SSH command to run to access the remote system.

Enter `ssh <username>@<ip address> -A` as the ssh command. If you are using ubuntu the username will be ubuntu.

![vs code install ssh extension](/posts/aws-cdk-cloud-dev-env/vscode-ssh-remote-server-address.png)

**OR**

2 - Using the command pallet (ctrl + shift + p).

Type in `>add new ssh host` then hit enter.

![vs code install ssh extension](/posts/aws-cdk-cloud-dev-env/vscode-ssh-add-connection-command-pallet.png)

Enter the ssh command to access the remote system, same as option 1. 

`ssh <username>@<ip address> -A`

![vs code install ssh extension](/posts/aws-cdk-cloud-dev-env/vscode-ssh-remote-server-address.png)

VS Code will ask where to save the SSH command, choose your home drive as the location to save it.

For example if your username is 'luigi' and you are using Linux save the config in /home/luigi/.ssh/config

![vs code ssh save config location](/posts/aws-cdk-cloud-dev-env/vscode-ssh-save-config-location.png)

## New CDK project quick start

Create a new folder with the name of the cdk project you want to make, then change into it.
```bash
mkdir cdk-project
cd "$_"
```


```bash
cdk init --language typescript
```

If you want to learn more about CDK there is an excelent workshop here. https://cdkworkshop.com/

## Optional extras

### RFDK (Render Farm Deployment Kit)

### Adding Terraform

### Git

Don't forget to set your git username and email

```
git config --global user.name "Your Name"
git config --global user.email "youremail@yourdomain.com"
```


## What next?

https://cdkworkshop.com/