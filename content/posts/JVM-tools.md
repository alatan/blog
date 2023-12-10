---  
title: "JVM调优相关工具与可调参数"  
date: 2018-05-06
weight: 70  
draft: false  
keywords: [""]  
description: "JVM调优相关工具与可调参数"  
tags: ["JVM"]
categories: ["JVM"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

## JDK工具包
* jps(JVM Process status tool)：JVM进程状态工具，查看进程基本信息
* jstat(JVM statistics monitoring tool)： JVM统计监控工具，查看堆，GC详细信息，可以通过它查看运行时堆信息的相关情况。
* jinfo(Java Configuration Info)：查看配置参数信息，支持部分参数运行时修改
  * jinfo -flags pid（打印虚拟机 VM 参数）
* jmap(Java Memory Map)：分析堆内存工具，dump堆内存快照
  * jmap -heap pid(显示Java堆详细信息：打印堆的摘要信息，包括使用的GC算法、堆配置信息和各内存区域内存使用信息)
* jhat(Java Heap Analysis Tool)：堆内存dump文件解析工具
* jstack(Java Stack Trace)：jstack是Java虚拟机自带的一种堆栈跟踪工具，用于生成java虚拟机当前时刻的线程快照。
* VisualVM(GUI)：性能分析可视化工具

## 第三方工具
### GCEasy
免费GC日志可视化分析Web工具：https://gceasy.io/
### MAT：Memory Analyzer Tool 可视化内存分析工具
可以快捷、有效地帮助我们找到内存泄露，减少内存消耗分析工具。能帮助你查找内存泄漏和减少内存消耗。
### GCViewer
开源的GC日志分析工具
### Arthas
Arthas 是一款线上监控诊断产品，通过全局视角实时查看应用 load、内存、gc、线程的状态信息，并能在不修改应用代码的情况下，对业务问题进行诊断，包括查看方法调用的出入参、异常，监测方法执行耗时，类加载信息等，大大提升线上问题排查效率。
#### Arthas 常见命令
* dashboard 
* jvm：查看当前 JVM 的信息
* thread：查看当前 JVM 的线程堆栈信息，
  * -b 选项可以一键检测死锁
  * -n 指定最忙的前N个线程并打印堆栈
* trace：方法内部调用路径，并输出方法路径上的每个节点上耗时，服务间调用时间过长时使用
* stack：输出当前方法被调用的调用路径
* Jad：反编译指定已加载类的源码，反编译便于理解业务
* logger：查看和修改 logger，可以动态更新日志级别
## JVM参数
JVM主要分三种：标准参数、非标准参数、不稳定参数
### 标准参数：不会随着JVM变化而变化
    * 以-开头，如：-version、-jar
    * 使用java -help查看
### 非标准参数：可能会随着JVM变化而变化，变化较小
    * 以-X开头，如：-Xmx、-Xms、-Xmn、-Xss
    * 使用java –X查看
#### 比较常见的非标准参数
1. -Xms堆最小值：默认值是总内存/64（且小于1G），默认情况下，当堆中可用内存小于40%（-XX: MinHeapFreeRatio调整）时，堆内存会开始增加，一直增加到-Xmx大小。
2. -Xmx堆最大值：默认值是总内存/64（且小于1G），如果Xms和Xmx都不设置，则两者大小会相同，默认情况下，当堆中可用内存大于70%时，堆内存会开始减少，一直减小到-Xms的大小；
3. -Xmn新生代内存：默认是整堆的1/3，包括Eden区和两个Survivor区的总和，写法如： -Xmn1024，-Xmn1024k，-Xmn1024m，-Xmn1g 。新生代按整堆的比例分配，所以，此值不建议设置！
4. -Xss线程栈内存：默认1M，一般来说是不需要改的。
5. 打印GC日志：-Xloggc:file与-verbose:gc功能类似，只是将每次GC事件的相关情况记录到一个文件中。
* -XX:NewRatio 设置新生代与老年代比值，-XX:NewRatio=4 表示新生代与老年代所占比例为1:4 ，新生代占比整个堆的五分之一。如果设置了-Xmn的情况下，该参数是不需要在设置的。 
* -XX:PermSize 设置持久代初始值，默认是物理内存的六十四分之一 
* -XX:MaxPermSize 设置持久代最大值，默认是物理内存的四分之一 
* -XX:MaxTenuringThreshold 新生代中对象存活次数，默认15。(若对象在eden区，经历一次MinorGC后还活着，则被移动到Survior区，年龄加1。以后，对象每次经历MinorGC，年龄都加1。达到阀值，则移入老年代) 
* -XX:SurvivorRatio Eden区与Subrvivor区大小的比值，如果设置为8，两个Subrvivor区与一个Eden区的比值为2:8，一个Survivor区占整个新生代的十分之一 
* -XX:+UseFastAccessorMethods 原始类型快速优化 
* -XX:+AggressiveOpts 编译速度加快 
* -XX:PretenureSizeThreshold 对象超过多大值时直接在老年代中分配

### 不稳定参数：也是非标准的参数，主要用于JVM调优与Debug
    * 以-XX开头，如： -XX:+UseG1GC 、 -XX:+UseParallelGC、 -XX:+PrintGCDetails
    * 分三类：性能参数、行为参数、和调试参数
#### 不稳定参数分为三类：
* 性能参数：用于JVM的性能调优和内存分配控制，如内存大小的设置；
* 行为参数：用于改变JVM的基础行为，如GC的方式和算法的选择；
* 调试参数：用于监控、打印、输出jvm的信息；
#### 常用的不稳定参数：
-XX:+UseSerialGC 配置串行收集器
-XX:+UseParallelGC 配置PS并行收集器
-XX:+UseParallelOldGC 配置PO并行收集器
-XX:+UseParNewGC 配置ParNew并行收集器
-XX:+UseConcMarkSweepGC 配置CMS并行收集器
-XX:+UseG1GC 配置G1并行收集器
-XX:+PrintGCDetails 配置开启GC日志打印
-XX:+PrintGCTimeStamps 配置开启打印GC时间戳
-XX:+PrintGCDateStamps 配置开启打印GC日期
-XX:+PrintHeapAtGC 配置开启在GC时，打印堆内存信息