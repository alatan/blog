---  
title: "JVM概览"  
date: 2018-05-01
weight: 70  
draft: false  
keywords: [""]  
description: "JVM概览"  
tags: ["JVM", "大纲"]
categories: ["JVM"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

## JVM概览
* 什么是JVM？广义上的JVM是指一种规范，狭义上的JVM指的是Hotspot类的虚拟机实现
* Java语言与JVM的关系：Java语言编写程序生成class字节码在JVM虚拟机里执行。其他语言也可以如Scala、Groovy
### 主要知识
1. JVM基本常识
2. 类加载系统
3. 运行时数据区（JVM 内存结构）
4. 一个对象的一生（出生、死亡与内涵）
5. GC垃圾收集器
6. JVM调优相关工具与可调参数
7. 调优实战案例

![](/images/jvm/JVM_Arch.png "JVM架构图")

## 参考文章
1. https://www.pdai.tech/md/java/jvm/java-jvm-jmm.html
2. https://www.pdai.tech/md/java/jvm/java-jvm-struct.html
3. https://www.pdai.tech/md/java/jvm/java-jvm-x-introduce.html