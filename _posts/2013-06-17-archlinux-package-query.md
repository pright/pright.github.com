---
layout: post
title: "ArchLinux已安装包查询"
description: ""
category: ""
tags: [ArchLinux, Linux]
---
{% include JB/setup %}

```
pacman -Qen | awk {'print $1'} > pac_native
pacman -Qem | awk {'print $1'} > pac_foreign
```
