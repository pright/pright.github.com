---
layout: post
title: "archLinux下debootstrap+schroot运行ubuntu10.04"
description: ""
category: ""
tags: [ArchLinux, Linux]
---
{% include JB/setup %}

## ArchLinux

#### 安装debootstrap及schroot

```
packer -S debootstrap schroot
```

#### debootstrap安装ubuntu

```
sudo debootstrap --arch=i386 lucid /chroot/ubuntu-10.04 http://mirrors.163.com/ubuntu/
sudo debootstrap --arch=amd64 lucid /chroot/ubuntu-10.04 http://mirrors.163.com/ubuntu/
```

#### 修改schroot配置
~~**_/etc/schroot/setup.d/15binfmt_** ^([1])~~
~~31行~~

~~elif ! which update-binfmts > /dev/null; then~~
~~修改为~~

~~elif ! which update-binfmts &> /dev/null; then~~

**_/etc/schroot/default/nssdatabases_**

* 注掉防止进入ubuntu时这些文件被arch下的同名文件覆盖导致用户和组丢失 ^([2])
* arch下没有/etc/networks ^([3])

```
#passwd
#shadow
#group
#gshadow
services
protocols
#networks
hosts
```

**_/etc/schroot/default/fstab_**

```
/opt         /opt         none rw,bind      0    0
/lib/modules /lib/modules none ro,bind      0    0       
```

**_/etc/schroot/schroot.conf_** ^([4])

```
[ubuntu]
description=Ubuntu-10.04
type=directory
directory=/chroot/ubuntu-10.04
users=pright
root-users=pright
aliases=lucid,default
```

#### schroot进入ubuntu

```
schroot -c lucid -u root -s /bin/bash
```

---

## Ubuntu

#### 防止无法更新包 ^([5])

```
dpkg-divert --local --rename --add /sbin/initctl
ln -s /bin/true /sbin/initctl
```

#### 添加缺少的user和group（id可能和下面不一致）

```
groupadd -g 19 log
groupmod -g 91 video
groupmod -g 92 audio
groupmod -g 94 floppy
groupadd -g 93 optical
groupadd -g 95 storage
groupadd -g 98 power
groupmod -g 14 uucp
groupadd -g 10 wheel
groupadd -g 90 network
groupadd -g 96 scanner
groupadd -g 998 bumblebee
adduser pright
usermod -G lp,wheel,video,audio,optical,floppy,storage,power pright
```

#### Oracle官网下载JDK 1.6
放置于/opt/java6目录

#### 配置JDK路径
**_/etc/profile.d/jdk6.sh_**

```
export J2SDKDIR=/opt/java6
export PATH=$PATH:/opt/java6/bin:/opt/java6/db/bin
export JAVA_HOME=/opt/java6
export DERBY_HOME=/opt/java6/db
```

**_/etc/profile.d/jre6.sh_**

```
export PATH=$PATH:/opt/java6/jre/bin
export JAVA_HOME=${JAVA_HOME:-/opt/java6/jre}
```

#### 添加10.04源
**_/etc/apt/sources.list_**

~~* 10.04已经移除sun jdk，使用8.04源里的安装 ^([6])~~

```
deb http://mirrors.163.com/ubuntu/ lucid main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ lucid-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ lucid-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ lucid-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ lucid-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ lucid main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ lucid-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ lucid-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ lucid-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ lucid-backports main restricted universe multiverse
```
~~# sun-java6-jdk~~
~~deb http://archive.ubuntu.com/ubuntu hardy-updates main multiverse~~
~~deb http://archive.ubuntu.com/ubuntu hardy main multiverse~~

#### 更新安装软件

```
apt-get update
apt-get upgrade
apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev libc6-dev lib32ncurses5-dev ia32-libs x11proto-core-dev libx11-dev lib32readline5-dev lib32z-dev libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown libxml2-utils xsltproc python uboot-mkimage lzma
apt-get install vim
```


---

*[1]: [Debian Bug report logs - #688304 Check for update-binfmts pollutes output because it does not redirect stderr from "which"](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=688304)*
*[2]: [schroot - frequently asked questions](http://manpages.ubuntu.com/manpages/natty/man7/schroot-faq.7.html)*
*[3]: [How to set up an Ubuntu chroot inside Arch Linux \[solved\]](https://bbs.archlinux.org/viewtopic.php?id=100039)*
*[4]: [schroot.conf — chroot definition file for schroot](http://manpages.ubuntu.com/manpages/hardy/man5/schroot.conf.5.html)*
*[5]: [unable to connect to upstart](http://www.ashang.org/2010/10/unable-to-connect-to-upstart.html)*
*[6]: [Canonical Will Remove Java From Ubuntu](http://news.softpedia.com/news/Canonical-Will-Remove-Java-From-Ubuntu-241147.shtml)*

