---
layout: post
title: "cscope问题"
description: ""
category: ""
tags: [cscope]
---
{% include JB/setup %}

cscope无法识别被如下宏包裹的函数

```
#ifdef __cplusplus
extern "C" {
#endif

void foo(void)
{
    ...
}

#ifdef __cplusplus
}
#endif
```

[bug页面](http://sourceforge.net/p/cscope/bugs/249/)
