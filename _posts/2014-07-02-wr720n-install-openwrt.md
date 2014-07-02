---
layout: post
title: "WR720N安装OpenWrt"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

* [这里](http://downloads.openwrt.org/snapshots/trunk/ar71xx/)下载最新镜像（factory为原厂固件安装用，sysupgrade为openwrt固件升级用）
* 路由器管理界面下安装镜像，需数分钟完成。
* 登录openwrt

```
telnet 192.168.1.1
```

* 配置网络

**_/etc/config/network_**

```
config interface 'lan'
        option ifname 'eth1'
        option type 'bridge'
        option proto 'static'
        option ipaddr '192.168.0.1'
        option netmask '255.255.255.0'
        option ip6assign '60'

config interface 'wan'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.1.180'
        option netmask '255.255.255.0'
        option gateway '192.168.1.1'
        option dns '192.168.1.1 8.8.4.4'
```

* 设置密码（OpenWrt将关闭telnet服务，以后只能通过ssh登录）

```
passwd
```

* 重启

```
reboot
```

* 重新连接Openwrt

```
ssh root@192.168.0.1
```

* 更新软件包列表

```
opkg update
```

* 安装luci管理界面并开启

```
opkg install luci luci-i18n-chinese
/etc/init.d/uhttpd enable
/etc/init.d/uhttpd start
```
之后即可用浏览器访问192.168.2.1进行管理。

