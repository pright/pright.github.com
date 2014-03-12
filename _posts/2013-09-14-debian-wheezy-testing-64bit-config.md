---
layout: post
title: "debian wheezy(testing)64位配置"
description: ""
category: ""
tags: [Debian, Linux]
---
{% include JB/setup %}

### 软件源
**_/etc/apt/source.list_**

``` 
# stable
deb http://mirrors.163.com/debian/ stable main non-free contrib
deb-src http://mirrors.163.com/debian/ stable main non-free contrib

# security
deb http://mirrors.163.com/debian-security stable/updates main non-free contrib
deb-src http://mirrors.163.com/debian-security stable/updates main non-free contrib

# stable-updates, previously known as 'volatile'
deb http://mirrors.163.com/debian/ stable-updates main contrib non-free
deb-src http://mirrors.163.com/debian/ stable-updates main contrib non-free

# backports
# deb http://mirrors.163.com/debian/ stable-backports main contrib non-free
# deb-src http://mirrors.163.com/debian/ stable-backports main contrib non-free

# testing
deb http://mirrors.163.com/debian testing main non-free contrib
deb-src http://mirrors.163.com/debian testing main non-free contrib

# testing-security
deb http://mirrors.163.com/debian-security testing/updates main contrib non-free
deb-src http://mirrors.163.com/debian-security testing/updates main contrib non-free

# testing-proposed-updates
deb http://mirrors.163.com/debian testing-proposed-updates main contrib non-free
deb-src http://mirrors.163.com/debian testing-proposed-updates main contrib non-free
```

*注：*
*1. testing源和backports源没必要同时使用。*
*2. 163等源存在更新不及时现象。*
*3. 可用如下命令找出最适合的源：*

```
apt-spy update
apt-spy -d testing -a asia -t 10
```


### 开启混合模式

```
dpkg --add-architecture i386
apt-get update
```

### 更新到testing

```
apt-get dist-upgrade
```

*注：*
*~~1. 更新导致gnome-core等软件包被删，暂不更新，下次直接通过testing dvd iso安装。~~*
*2. testing源无法安装ia32-libs。按官方说明，ia32-libs只是过渡用，以后将不需要。*

### ~~安装32位库文件~~
~~apt-get install ia32-libs ia32-libs-gtk~~
 
### 安装软件

```
apt-get install python-openssl python-pip python-scrapy python-flask python-m2crypto python-gdata python-iso8601 python-autopep8 ipython libnss3-tools python-vte bison apt-transport-https libxml2-utils xsltproc htop strace ltrace xtrace build-essential libtool valgrind dkms gcc g++ make automake cmake openjdk-6-jdk android-tools-adb android-tools-fastboot subversion git git-email git-flow gitk tig ctags cscope vim vim-gnome ack-grep zsh tmux autojump tftp samba tftpd nfs-kernel-server sshfs rsync minicom rabbitvcs-nautilus chromium chromium-l10n icedove icedove-l10n-zh-cn wireshark tcpdump tcptrace tcptrack iptraf udpcast iperf mtr-tiny socat gdb meld flashplugin-nonfree icedtea-6-plugin ntpdate uboot-mkimage exaile swig tstools dvbsnoop ranger markdown mutt offlineimap msmtp libnotify-bin gcc-multilib g++-multilib gcc-4.4-multilib g++-4.4-multilib zlib1g zlib1g-dev lib32z1-dev zip gperf flex curl tofrodos lib32ncurses5-dev libglib2.0-0:i386 libpng12-0:i386 libsm6:i386 libxrender1:i386 libfontconfig1:i386 glances nautilus-actions rake ruby1.9.3 ruby-switch xclip xsel libjpeg62
```

~~apt-get install nautilus-open-terminal~~

goagent：https://code.google.com/p/goagent/ 
wineqq：http://www.longene.org/forum/viewtopic.php?t=4700
xunlei-lixian：https://github.com/iambus/xunlei-lixian
dropbox：https://www.dropbox.com/install?os=lnx
yunio：https://www.yunio.com/
sogoupinyin：http://packages.linuxdeepin.com/deepin/pool/non-free/f/fcitx-sogoupinyin-release/ （需要更新testing源里的libc6、fcitx-bin等）
~~gnome-shell-google-calendar：https://github.com/vintitres/gnome-shell-google-calendar~~
ADT Bundle：https://developer.android.com/sdk/index.html
virtualbox：https://www.virtualbox.org/wiki/Linux_Downloads
wps：http://community.wps.cn/download/

*注：*
_1. icedove即debian重编译过的thunderbird_
_2. vim的\*和+寄存器需要安装vim-gnome_
_3. 部分在线flash视频无法用gnash播放，需要安装flashplugin-nonfree_

### 配置
#### root用户
  * sudo

    ```
    adduser pright sudo
    ```

  * 用NetworkManager接管有线网络接口

    ```
    sed -i 's/allow-hotplug eth0/#allow-hotplug eth0/' /etc/network/interfaces
    sed -i 's/iface eth0 inet dhcp/#iface ethn0 inet dhcp/' /etc/network/interfaces
    service network-manager restart
    ```

  * 修改默认editor

    ```
    update-alternatives --config editor
    ```
    选择vim.basic

  * minicom使用ttyS\*设备权限

    ```
    adduser pright dialout
    ```

  * 安装Monaco字体

    ```
    mkdir -p /usr/share/fonts/truetype/custom
    cp Monaco.ttf /usr/share/fonts/truetype/custom
    fc-cache -f -v
    ```

  * 关闭pc喇叭

    ```
    echo "blacklist pcspkr" > /etc/modprobe.d/nobeep.conf
    ```

  * wireshark非root用户权限

    ```
    dpkg-reconfigure wireshark-common
    adduser pright wireshark
    ```

  * virtualbox

    ```
    echo "deb http://download.virtualbox.org/virtualbox/debian wheezy contrib" > /etc/apt/sources.list.d/virtualbox.list
    wget -q http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc -O- | sudo apt-key add -
    ```

    ~~echo "vboxdrv" >> /etc/modules~~

  * virtualbox使用usb
    
    ~~adduser pright vboxusers~~
    ~~echo "none /proc/bus/usb usbfs dvgid=\`cat /etc/group | grep vboxusers | cut -d ':' -f 3\`,devmode=644 0 0" >> /etc/fstab~~

  * dropbox

    ```
    sed -i 's/http:/https:/' /etc/apt/sources.list.d/dropbox.list
    ```

  * rar解压缩中文乱码（使用unrar替代rar）

    ```
    apt-get remove rar
    apt-get install unrar
    ```

  * 解决找不到gnome-keyring-pkcs11.so的问题：
    p11-kit: couldn't load module: /usr/lib/i386-linux-gnu/pkcs11/gnome-keyring-pkcs11.so: /usr/lib/i386-linux-gnu/pkcs11/gnome-keyring-pkcs11.so: cannot open shared object file: No such file or directory

    ```
    apt-get download gnome-keyring:i386
    dpkg -x gnome-keyring_3.8.2-2_i386.deb gnome-keyring
    cp -r gnome-keyring/usr/lib/i386-linux-gnu/pkcs11/ /usr/lib/i386-linux-gnu/
    ```

  * ctrl+alt+backspace开启

    ```
    dpkg-reconfigure keyboard-configuration
    ```

  * 解决不能正常挂载u盘

    ```
    sed -i 's/\/dev\/sr0/#\/dev\/sr0/g' /etc/fstab
    sed -i 's/\/dev\/sdb1/#\/dev\/sdb1/g' /etc/fstab
    ```

  * android ics编译
    \# gcc/g++

    ```
    ln -sf /usr/bin/gcc-4.4 gcc
    ln -sf /usr/bin/g++-4.4 g++
    ```
    
    \# Workaround for now

    ```
    ln -s /usr/include/x86_64-linux-gnu/zconf.h /usr/include
    ```

    \# jdk

    ```
    apt-get install java-package
    ```

    http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase6-419409.html 下载jdk6

    ```
    make-jpkg ./jdk-6u45-linux-x64.bin
    dpkg -i oracle-java6-jdk_6u45_amd64.deb
    ln -s /usr/lib/jvm/jdk-6-oracle-x64 /usr/lib/jvm/java-6-sun
    ln -s .jdk-6-oracle-x64.jinfo .java-6-sun.jinfo
    update-java-alternatives -s java-6-sun
    ```

  * rabbitvcs-nautilus无法显示问题(Workaround)
    https://code.google.com/p/rabbitvcs/issues/detail?id=803

    ```
    ln -sf /usr/lib/x86_64-linux-gnu/libpython2.7.so.1.0 /usr/lib/
    wget http://rabbitvcs.googlecode.com/svn/trunk/clients/nautilus-3.0/RabbitVCS.py /usr/share/nautilus-python/extensions
    ```

#### pright用户
  * grml-zsh-config

    ```
    wget -O ~/.zshrc http://git.grml.org/f/grml-etc-core/etc/zsh/zshrc
    ```

    http://grml.org/zsh

  * 注掉grml-zsh-config的alias，防止和autojump冲突

    ```
    sed -i 's/\^alias j=/#alias j=/' ~/.zshrc
    ```

  * 打开autojump

    ```
    echo "[[ -f /usr/share/autojump/autojump.zsh ]] && source /usr/share/autojump/autojump.zsh" > ~/.zshrc.local
    ```

  * 修改shell为zsh

    ```
    chsh -s /usr/bin/zsh
    ```

  * git
    
    ```
    git clone https://github.com/pright/gitconfig ~/github/gitconfig
    cd ~/github/gitconfig
    ./install
    ```

    （git 1.7.11以前修改[push] default = matching）

  * xterm配置

    ```
    git clone https://github.com/pright/xresources ~/github/xresources
    cd ~/github/xresources
    ./install
    ```

  * xterm设为默认终端
    \# 以下对testing源的nautilus-open-terminal不起作用，通过nautilus-actions自行设定

    ```
    gsettings set org.gnome.desktop.default-applications.terminal exec xterm
    sudo update-alternatives --config x-terminal-emulator
    ```

    应用程序->系统工具->主菜单->附件，修改终端命令行为xterm

  * nautilus在当前位置打开终端

    ```
    nautilus-actions-config-tool
    ```

    新建动作“在终端中打开(E)” ，选中“显示选择右键菜单中的项目”和“Display item in location context menu”。命令路径中填“/usr/bin/xterm”，参数放空，工作目录填“%d/%b”。

  * vim

    ```
    git clone https://github.com/pright/vimrc ~/github/vimrc
    cd ~/github/vimrc
    ./install
    git clone https://github.com/pright/utils ~/github/utils
    cd ~/github/utils
    ./install
    git clone http://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
    ```

  * tmux

    ```
    git clone https://github.com/pright/tmuxconf ~/github/tmuxconf
    cd ~/github/tmuxconf
    ./install
    ```

  * 添加goagent自启动

    ```
    mkdir -p ~/.config/autostart 
    /home/pright/goagent/local/addto-startup.py
    ```

  * xunlei-lixian

    ```
    ln -sf ~/github/xunlei-lixian/lixian_cli.py ~/bin/lx
    ```

  * ~/bin路径

    ```
    echo "export PATH=$PATH:~/bin" >> ~/.zshrc.local 
    ```

  * 设置日历默认应用

    ```
    gsettings set org.gnome.desktop.default-applications.office.calendar exec "chromium 'https://www.google.com/calendar'"
    ```

  * gnome-shell-google-calendar~~
    \# gnome3.8改用oauth2,此插件已无效  
    ~~git clone https://github.com/vintitres/gnome-shell-google-calendar ~/github/gnome-shell-google-calendar gnome-session-properties~~
    ~~命令："python /home/pright/github/gnome-shell-google-calendar/gnome-shell-google-calendar.py --account pright.xj@gmail.com"~~

  * gnome-shell设置

    ```
    sudo apt-get remove gnome-shell-extensions
    ```

    *注：当前testing源内的Auto Move Windows扩展有问题，使用官方最新的扩展*

    https://extensions.gnome.org 搜索安装Advanced Settings in UserMenu扩展、Alternative Status Menu扩展、Auto Move Windows扩展、Coverflow Alt-Tab扩展、Places Status Indicator扩展、Removable Drive Menu扩展、Remove Accessibility扩展、SystemMonitor扩展、Weather扩展
    
    ```
    gnome-tweak-tool
    ```

    alt+f2，r
    Shell->Show date in clock

  * gnome-shell插件设置

    ```
    gnome-shell-extension-prefs
    ```

  * exaile豆瓣fm插件

    ~~sudo apt-get -t wheezy-backports install exaile~~

    ```
    git clone https://github.com/pright/exaile-doubanfm-plugin ~/.local/share/exaile/plugins/doubanfm
    ```

  * thunderbird回复内容前置
    工具–>首选项–>高级–>高级配置–>配置编辑器
    mail.identity.default.reply_on_top值改为1

  * jekyll
    http://jekyllbootstrap.com/usage/jekyll-quick-start.html
    \# 本地运行

    ```
    sudo gem install github-pages
    jekyll serve
    ```

  * gitignore

    ```
    sudo gem install gitignore
    ```

