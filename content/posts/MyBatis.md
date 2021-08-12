---  
title: " MyBatis总结"
description: "MyBatis总结"
keywords: ["MyBatis"]
date: 2019-01-01
author: "默哥"
weight: 70
draft: false

categories: ["数据库"]
tags: ["数据库", "MyBatis", "框架", "大纲"]  
toc: 
    auto: false
---

![](/images/mybatis/mybatis.png "MyBatis框架整体设计")

## 接口层-和数据库交互的方式
MyBatis和数据库的交互有两种方式：
* 使用传统的MyBatis提供的API；
* 使用Mapper接口；
