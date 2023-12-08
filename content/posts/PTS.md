---  
title: "性能测试"  
date: 2020-05-10
weight: 70  
draft: false  
keywords: ["性能测试"]  
description: "性能测试"  
tags: [ "性能测试"]  
categories: ["知识体系"]  
author: "默哥"  

lightgallery: true
toc:
  auto: false
---  ˚
![](/images/test/test.png "问题排查")

### 测试指标
* 业务指标：如并发用户数、TPS（系统每秒处理事务数）、成功率、响应时间。
* 资源指标：如CPU资源利用率、内存利用率、I/O、内核参数（信号量、打开文件数）等。
* 应用指标：如空闲线程数、数据库连接数、GC/FULL GC次数、函数耗时等。
* 前端指标：如页面加载时间、网络时间（DNS、连接时间、传输时间等）。

### 工具
* APM工具（如ARMS）进行中间件、数据库、应用层面的问题定位。