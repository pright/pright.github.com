---
layout: post
title: "OpenGrok配置"
description: ""
category: ""
tags: [OpenGrok]
---
{% include JB/setup %}

* [这里](http://opengrok.github.io/OpenGrok)下载最新版本OpenGrok，目前为0.11.1
* 解压到任意目录，以下操作以/opt/opengrok目录为例
* 修改/opt/opengrok/bin/OpenGrok实例路径为/opt/opengrok以及调整JAVA内存使用防止解析大文件出错

```
sed -i 's/OPENGROK_INSTANCE_BASE:-\/var\/opengrok/OPENGROK_INSTANCE_BASE:-\/opt\/opengrok/' OpenGrok
sed -i 's/JAVA_OPTS:--Xmx2048m/JAVA_OPTS:--Xmx4096m -Xms2048m/' OpenGrok
```
* 创建/opt/opengrok/src目录并添加需建立索引的代码目录
* 安装tomcat6（目前版本tomcat7不能直接支持，需要修改OPENGROK_TOMCAT_BASE）并开启服务
* 修改tomcat6默认端口号

```
sed -i 's/<Connector port="8080"/<Connector port="8088"/' /etc/tomcat6/server.xml
```
* 部署OpenGrok

```
/opt/opengrok/bin/OpenGrok deploy
```
* 生成索引文件

```
/opt/opengrok/bin/OpenGrok index
```
* 增加定期更新索引的crontab

```
crontab -e
0 2 * * * /opt/opengrok/bin/OpenGrok update
```
* 访问[页面](http://127.0.0.1:8080/source)即可浏览代码


