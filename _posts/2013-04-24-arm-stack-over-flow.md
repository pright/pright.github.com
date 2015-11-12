---
layout: post
title: "arm平台堆栈溢出分析"
description: ""
category: ""
tags: [ARM, Programming]
---
{% include JB/setup %}

* 检查异常处调用函数是否存在堆栈溢出。

* 查看内存映射，最后一行为主线程堆栈起始地址及大小，线程堆栈地址需要在栈帧底部查看$sp确定（从内存映射可查到对应的堆栈起始地址和大小）。
(gdb) info proc mapping
0xbea8f000 0xbeab0000 0x21000 0 [stack]

* 检查异常线程堆栈是否溢出。
检查异常处$sp，和栈帧底部$sp比较是否超出大小（是否小于对应堆栈起始地址）。
(gdb) print $sp

* 检查异常线程堆栈下方堆栈空间是否溢出。
根据内存映射确认下方堆栈空间大小，可通过gdb条件断点定位线程创建函数，获取线程入口，单步跟踪确认内存是否溢出。

* 检查函数堆栈使用情况
按使用堆栈大小排列显示函数

```
arm-hisiv200-linux-objdump -d --prefix-address ./cyhisiapp > dump
cat dump | grep 'sub.*sp.*sp' | awk -F, {'print $3"  "$1'} | sed 's/#//' | sort -rn > out  
```

* 调整线程堆栈分配。

