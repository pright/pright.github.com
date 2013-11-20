---
layout: post
title: "gitblit创建空版本库显示Internal error问题的解决"
description: ""
category: ""
tags: [Gitblit]
---
{% include JB/setup %}

中文界面的空版本库页面文字编码导致。

解决办法如下：
1. 解压缩gitblit.jar。
2. 转换com/gitblit/wicket/pages/emptyRepositoryPage_zh_cn.html文件编码为ANSI并保存。
3. 重新启动gitblit即可。
