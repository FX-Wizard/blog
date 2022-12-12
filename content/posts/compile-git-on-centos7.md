+++
title = "Latest Git on Centos7"
date = "2022-12-09T12:49:13+11:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["linux", "git"]
keywords = ["git", "centos7", "linux", "compile"]
description = "How to compile latest version of Git on CentOS 7"
showFullContent = false
readingTime = false
hideComments = false
color = "red" #color from the theme settings
+++

Centos 7 comes with a really old version of Git that is missing handy features like hooks.

Here is how to compile the latest version of Git on CentOS 7 yourself.



Start with updating the system.
```bash
sudo yum -y update
```

Install dnf package manager to get all the improvements over yum.
```bash
sudo yum -y install dnf
```

Add EPEL repository to access packages that are not in the standard repositories.
```bash
sudo dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

Install build dependencies.
```bash
sudo dnf -y install asciidoc curl-devel dh-autoreconf docbook2X expat-devel gettext-devel openssl-devel perl-devel xmlto zlib-devel
sudo dnf -y install wget
```

Dowload source, verify and extract.

*As of writing the latest version of git is `2.38.1` however there may have been a newer version released by the time you are reading this. You can go to [this webpage](https://mirrors.edge.kernel.org/pub/software/scm/git/) to check if there is a newer version.*

```bash
export VERSION=2.38.1
wget -L https://mirrors.edge.kernel.org/pub/software/scm/git/git-$VERSION.tar.xz
wget -L https://mirrors.edge.kernel.org/pub/software/scm/git/git-$VERSION.tar.sign
unxz git-$VERSION.tar.xz
gpg --keyserver-options auto-key-retrieve $("git-$VERSION.tar.sign")
tar -xf git-$VERSION.tar
cd git-$VERSION
```

# Build
```bash
make configure
```

Set output to home drive
```bash
./configure --prefix=$HOME/.local/$(gcc -dumpmachine)
```

Run make
```bash
make all -j$(nproc)
make install
```