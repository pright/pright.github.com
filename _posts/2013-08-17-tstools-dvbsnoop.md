---
layout: post
title: "tstools/dvbsnoop使用"
description: ""
category: ""
tags: [DVB]
---
{% include JB/setup %}

* 查看foo.ts文件信息

  ```
  tsinfo foo.ts
  ```

* 查看foo.ts文件码率

  ```
  tsreport foo.ts -t
  ```

* 输出foo.ts文件中所有的PID

  ```
  dvbsnoop -if foo.ts -s ts -pd 3 -nph | grep '^PID: ' | sort -k2n | uniq  
  ```

* 输出foo.ts文件中pid为PID的数据

  ```
  tsreport foo.ts -justpid PID -m 1
  ```

* 输出foo.ts文件中pid为PID的si/pes数据
  
  ```
  dvbsnoop -if foo.ts -s ts -tssubdecode -pd 4 -nph PID
  ```

* 输出foo.ts文件中pid为PID的连续计数值

  ```
  tsreport foo.ts -cnt PID
  cat continuity_counter.txt
  ```

