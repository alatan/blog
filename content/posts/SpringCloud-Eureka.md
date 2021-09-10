---  
title: "Spring Cloud Eureka 解析"
description: "Spring Cloud Eureka"
keywords: ["SpringCloud"]
date: 2020-04-03
author: "默哥"
weight: 70
draft: false

categories: ["SpringCloud"]
tags: ["SpringCloud", "大纲"]  
toc: 
    auto: false
---

先来一波问题，然后看看Enueka是通过什么方式处理的。
* Eureka注册中心使用什么样的方式来储存各个服务注册时发送过来的机器地址和端口号？
* 各个服务找Eureka Server拉取注册表的时候，是什么样的频率？
* 各个服务是如何拉取注册表的？
* 对于一个有几百个服务，部署上千台机器的大型分布式系统来说，这套系统会对Eureka Server造成多大的访问压力？
* Eureka Server从技术层面是如何抗住日千万级访问量的？


## 参考文章 
* https://www.toutiao.com/i6621407284914291213/?wid=1631243211323