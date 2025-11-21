+++
title = "Automate AMI Creation With Packer"
date = "2025-11-20T15:50:52+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["PACKER", "AUTOMATION", "AWS", "CLOUD", "DEVOPS", "INFRASTRUCTURE-AS-CODE"]
keywords = ["packer", "aws", "ami automation", "infrastructure as code", "aws ami", "packer tutorial", "hashicorp packer", "packer aws", "ami builder", "infrastructure automation", "devops automation", "gpu workstation", "nice dcv", "nvidia drivers", "cloud workstation", "ubuntu ami", "ec2 automation", "aws infrastructure", "iac", "packer guide", "terraform"]
description = "Learn how to automate AWS AMI creation with Packer instead of clicking through the console manually. This step-by-step guide shows you how to build consistent, repeatable infrastructure using Infrastructure as Code—complete with GPU drivers, NICE DCV, and custom software installations."
showFullContent = false
readingTime = true
hideComments = false
color = "orange"
+++

I've covered creating images for [3D workstations](ubuntu-24.04-3d-ws-aws) and [render nodes](deadline-linux-worker), but that was a very manual process. So I've written this guide to show how to automate the process. In this guide we'll use **Packer**, an open-source tool that lets you automate the entire process of creating custom AMIs (Amazon Machine Images). By the end, you'll understand why automation is a game-changer for teams that need consistent, repeatable infrastructure.


Example repos on GitHub
- [3D Workstation](https://github.com/FX-Wizard/packer-cloud-workstation-builder)
- [Deadline Render Worker](https://github.com/FX-Wizard/packer-deadline-worker-builder)

### Why Packer?

*Why Not Just Click Around in the AWS Console?*

The Console is a good option if you only want to deploy something once, but if you have to deploy something more than once, then automating the process has its benefits:

- **Consistency**: Manual processes are error-prone. One person might install NVIDIA drivers version X, another person installs version Y. Packer ensures everyone gets the exact same setup.
- **Scalability**: Need to create 50 AMIs across different regions? Packer can do it automatically.
- **Reproducibility**: Six months from now, you'll forget exactly which steps you took. Your Packer config is your documentation.
- **Speed**: Let Packer handle the heavy lifting while you grab coffee.


### What We Will Build

Using the [3D Workstation](https://github.com/FX-Wizard/packer-cloud-workstation-builder) example, let's create a base AMI for a 3D artist or CAD engineer:

- A full Ubuntu desktop environment with GPU acceleration
- NVIDIA GPU drivers (optimized for compute instances)
- NICE DCV remote desktop access (because cloud workstations need remote access)
- Pre-installed software for 3D rendering and CAD work
- All the system libraries and dependencies these tools require


## Anatomy Of The Project

### Packer Config

Let's look at what makes this all work. At the heart of everything is the Packer configuration file:

```hcl
# image.pkr.hcl - The blueprint for our AMI
```

The Packer configuration file is basically a recipe that tells Packer:

1. Where to build (AWS)
2. What to start with (a base Ubuntu AMI)
3. What commands to run to configure it
4. What to name the final image

Here's the general flow:

1. **Source Block**: This defines the starting point. We're using a standard Ubuntu 22.04 LTS AMI and specifying the instance type (important for GPU workloads!).

Example:
```hcl
source "amazon-ebs" "ubuntu" {
   ...
}
```

2. **Build Block**: This is where the magic happens. It specifies which provisioners to run and in what order. We use shell scripts to:

   - Update the system packages
   - Install NVIDIA GPU drivers
   - Set up a graphical desktop environment
   - Install and configure NICE DCV for remote access
   - Install CAD and 3D rendering software

Example:
```hcl
build {
  name = "${var.name_prefix}-build"
  ...
  provisioner "shell" {
   <can add your shell scripts here>
  }
}
```

## Installation Scripts

The Packer configuration itself is pretty lean because the heavy lifting is done by shell scripts. Let's break them down:

### 1. **update-system.sh** - Foundation First

This script handles the basics:

- Updates all system packages
- Installs build tools and dependencies needed for GPU driver compilation
- Sets up the foundation for everything that comes next

Why this matters: GPU drivers need to compile against your kernel, so having the right build tools is essential.

### 2. **install-nvidia-drivers.sh** - The GPU Brain

Having up-to-date drivers is critical for any GPU workload. The script:

- Downloads and installs NVIDIA GPU drivers optimized for compute
- Installs CUDA libraries
- Verifies the installation with `nvidia-smi`

For 3D rendering and CAD work, you *need* this. Without it, your GPU is just an expensive paperweight.

### 3. **install-desktop.sh** - Making It Graphical

By default, AWS Ubuntu instances are headless. This script installs:

- A lightweight desktop environment (GNOME)
- X11 libraries
- Desktop utilities

### 4. **install-dcv.sh** - Remote Access Magic

NICE DCV (Desktop Cloud Visualization) is AWS's remote desktop protocol. It's optimized for:

- Low latency graphics streaming
- GPU acceleration over the network
- Security (encrypted connections)

This script sets up DCV so users can connect to their workstations from anywhere. Perfect for distributed teams or on-the-go work.

### 5. **install-software.sh** - The Creative Tools

This is where you'd add your specific applications:

- Blender for 3D rendering
- FreeCAD for design work
- Video editing software
- Any other tools your team needs

You can customize this to match your workflows.

### Adding your own scripts
You can easily extend the workstation image by adding custom installation scripts. The build process is defined in `image.pkr.hcl` and follows a specific sequence.

### Adding Custom Installation Scripts

1. **Create a new shell script** in the project root directory (e.g., `install-custom-software.sh`):

```bash
#!/bin/bash
set -e

echo "Installing custom software..."

# Add your installation commands here
# Example: Install Blender

# Update package manager
sudo apt-get update

# Install Blender
sudo apt-get install -y blender

echo "Custom software installation completed."
```

2. **Make the script executable**:

```bash
chmod +x install-custom-software.sh
```

3. **Add a provisioner to `image.pkr.hcl`** in the `build` block:

```hcl
provisioner "shell" {
  script       = "./install-custom-software.sh"
  max_retries  = 3
}
```

> Note: scripts execute in order, some scripts are depended on previous installations.


## How to Build Your Packer Project

### Prerequisites

- AWS account with appropriate permissions
- Packer installed locally (or run in a container)
- AWS CLI configured with credentials
- A basic understanding of how AMIs work

### Step 1: Customize the Configuration

Open `image.pkr.hcl` and adjust:

- The base AMI (if you want a different Ubuntu version)
- Instance type (G instances are for GPU workloads)
- Region (where your workstations will live)
- Output AMI name

### Step 2: Validate Your Configuration

```bash
packer validate image.pkr.hcl
```

This checks for syntax errors and configuration issues before you spin up any infrastructure.

### Step 3: Build the AMI

```bash
packer build image.pkr.hcl
```

Now Packer will:

1. Launch an EC2 instance with your specified configuration
2. Run all the provisioner scripts in order
3. Create a snapshot of the configured instance
4. Register it as an AMI
5. Clean up the temporary instance

Depending on what you're installing, this might take 10-20 minutes. Grab that coffee now.

### Step 4: Launch Instances from Your AMI

Once the build completes, you'll see an AMI ID in the output. Use that ID in the AWS console, CLI, or your Infrastructure-as-Code tools (Terraform, CloudFormation, etc.) to launch instances.


## Tips & Tricks

### 1. **Start Small, Iterate Fast**

Don't try to build the perfect AMI on your first try. Start with just the GPU drivers and desktop, test it, then add more software incrementally. This makes debugging easier.

### 2. **Test in a Small Instance Type First**

Use a smaller instance for testing builds (faster, cheaper), then scale to your production instance type once everything works.

### 3. **Version Your AMIs**

Include a version number or date in your AMI name. It helps you track what's installed in each image:

```
workstation-v1.0-2025-01-15
workstation-v1.1-2025-02-01
```

### 4. **Keep Scripts Modular**

Each script should have a single responsibility. This makes it easier to test, debug, modify, and reuse individual components.

### 5. **Add Verification Steps**

In your scripts, verify that installations succeeded:

```bash
# Check that GPU drivers installed correctly
nvidia-smi || exit 1

# Verify DCV is running
systemctl is-active dcvserver || exit 1
```

If something fails, Packer will stop the build and you'll know exactly where things went wrong.

### 6. **Document Everything**

Comments in your Packer config and scripts are your friends. 

- Use git to manage different image versions and easily roll if an update breaks
- Add comments to scripts to explain how they work
- Enables team members to quickly understand infrastructure without tribal knowledge
- Provides audit trail for compliance and security reviews

Future-you will appreciate it.

## Troubleshooting

**Build fails during GPU driver installation?**

- Check that you're using a GPU instance type (g4dn, g4ad, etc.)
- Verify the NVIDIA driver version is compatible with your CUDA version

**DCV isn't connecting?**

- Check security group rules allow port 8443 (or your configured DCV port)
- Verify the DCV service started successfully

**Software isn't installing?**

- Check internet connectivity in your provisioner scripts
- Make sure the EC2 build instance has the correct security groups assigned
- Some repos might have regional variations—adjust URLs as needed

---

## References & Resources

- [Packer Documentation](https://www.packer.io/docs)
- [AWS Compute Optimizer for GPU instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/accelerated-computing-instances.html)
- [NICE DCV Documentation](https://docs.aws.amazon.com/dcv/)
- [NVIDIA CUDA and Driver Documentation](https://developer.nvidia.com/cuda-toolkit)
