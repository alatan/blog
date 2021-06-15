---  
title: "JVM详解"  
date: 2018-04-20
weight: 70  
draft: false  
keywords: [""]  
description: "JVM详解"  
tags: ["JVM", "大纲"]
categories: ["Java基础"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

![](/images/jvm/jvm-overview.png "JVM概览")

## Class文件的结构属性
![](/images/jvm/class-struct.png "类的结构")


## Java类加载机制
### 类的生命周期
![](/images/jvm/class-life.png "类的生命周期")

### 类加载器
* 启动类加载器: Bootstrap ClassLoader，负责加载存放在JDK\jre\lib(JDK代表JDK的安装目录，下同)下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库(如rt.jar，所有的java.*开头的类均被Bootstrap ClassLoader加载)。启动类加载器是无法被Java程序直接引用的。 
* 扩展类加载器: Extension ClassLoader，该加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载JDK\jre\lib\ext目录中，或者由java.ext.dirs系统变量指定的路径中的所有类库(如javax.*开头的类)，开发者可以直接使用扩展类加载器。 
* 应用程序类加载器: Application ClassLoader，该类加载器由sun.misc.Launcher$AppClassLoader来实现，它负责加载用户类路径(ClassPath)所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

### JVM类加载机制 
* 全盘负责，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入 
* 父类委托，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类 
* 缓存机制，缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效 
* 双亲委派机制, 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

## JVM 内存结构
![](/images/jvm/jms.jpg "JVM内存结构")

* 栈是运行时的单位，而堆是存储的单位。（**栈解决程序的运行问题，即程序如何执行，或者说如何处理数据。堆解决的是数据存储的问题，即数据怎么放、放在哪。**）
* Java虚拟机栈用于管理Java方法的调用，而本地方法栈用于管理本地方法的调用。

### 程序计数器


### 虚拟机栈


### 本地方法栈


### 堆内存

## JVM内存模型(JMM)

## Java垃圾回收

## JVM参调优

## JVM分析和问题排查


## 参考文章
1. https://www.pdai.tech/md/java/jvm/java-jvm-jmm.html