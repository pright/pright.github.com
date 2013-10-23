---
layout: post
title: "ArchLinux安装记录"
description: ""
category: ""
tags: [ArchLinux, Linux]
---
{% include JB/setup %}

最近很背，连续重装系统，再次把arch弄垮了。这里记下安装记录，给下次安装备用。

* package选择base-dev，openssh，nfs-utils，nfsidmap，sudo

* 镜像

  ```
  cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist_bak
  sed -i 's/^#Server/Server/g' /etc/pacman.d/mirrorlist_bak
  rankmirrors -n 6 /etc/pacman.d/mirrorlist_bak > /etc/pacman.d/mirrorlist
  pacman -Syy
  pacman -S powerpill
  powerpill -Syu 
  ```

* aur
  **_/etc/pacman.conf_**
  
  ```
  [archlinuxfr]
  Server = http://repo.archlinux.fr/i686
  ```
  
  ```
  powerpill -S yaourt
  yaourt -S packer
  ```

* 自动补全

  ```
  powerpill -S bash-completion
  ```

* 用户
  
  ```
  groupadd pright
  adduser
  ```
  
  增加group: audio,floppy,lp,optical,power,storage,video,wheel
  
  ```
  visudo
  ```
  
  增加如下
  
  ```
  %wheel ALL=(ALL) ALL
  ```

* 安装软件
  
  ```
  su pright
  sudo powerpill -S vim samba tftp-hpa subversion git
  packer ranger
  ```

* nfs
  **_/etc/exports_**
  
  ```
  /home/pright/share *(rw,sync,nohide)
  ```
  
  ```
  sudo /etc/rc.d/nfs-server restart
  ```

* samba
  
  ```
  sudo cp /etc/samba/smb.conf.default /etc/samba/smb.conf
  ```
  
  **_/etc/samba/smb.conf_**
  
  ```
  [home]
  path = /home
  valid users = pright
  public = no
  writable = yes
  printable = no
  create mask = 0765
  ```
  
  ```
  testparm
  sudo smbpasswd -a pright
  sudo /etc/rc.d/samba restart
  ```

* tftp
  **_/etc/conf.d/tftpd_**
  
  ```
  TFTPD_ARGS="-l -s -c /var/tftpboot"
  sudo chmod 777 /var/tftpboot
  sudo /etc/rc.d/tftpd restart
  ```

* 主机名解析
  **_/etc/nsswitch.conf_**
  
  ```
  hosts: files dns wins
  ```

* 守护进程
  **_/etc/rc.conf_**
  
  ```
  DAEMONS=(syslog-ng network netfs crond @sshd @samba @nfs-server @tftpd)
  ```
