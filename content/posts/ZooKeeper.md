---  
title: "ZooKeeper"  
date: 2020-03-15
weight: 70  
draft: false  
keywords: ["分布式"]  
description: "ZooKeeper"  
tags: [ "分布式"]  
categories: ["分布式"]  
author: "默哥"  

toc:
  auto: false
---  
* ZooKeeper主要服务于分布式系统，可以用ZooKeeper来做：统一配置管理、统一命名服务、分布式锁、集群管理。
* 使用分布式系统就无法避免对节点管理的问题(需要实时感知节点的状态、对节点进行统一管理等等)，而由于这些问题处理起来可能相对麻烦和提高了系统的复杂性，ZooKeeper作为一个能够通用解决这些问题的中间件就应运而生了。


## 参考文章
* https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453140996&idx=1&sn=b2391f3eb780529020ace3a4c4357bda&chksm=8cf2d487bb855d916bca80d709ed8e22457af52d1746e093fe8d4e3d4d095dbcc767e68dd1c1&scene=21