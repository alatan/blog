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


# JVM 调优参数
### JVM参数
* -Xms 堆最小值 
* -Xmx 堆最大堆值。-Xms与-Xmx 的单位默认字节都是以k、m做单位的。 通常这两个配置参数相等，避免每次空间不足，动态扩容带来的影响。 
* -Xmn 新生代大小 
* -Xss 每个线程池的堆栈大小。
  在jdk5以上的版本，每个线程堆栈大小为1m，jdk5以前的版本是每个线程池大小为256k。一般在相同物理内存下，如果减少－xss值会产生更大的线程数，但不同的操作系统对进程内线程数是有限制的，是不能无限生成。 
* -XX:NewRatio 设置新生代与老年代比值，-XX:NewRatio=4 表示新生代与老年代所占比例为1:4 ，新生代占比整个堆的五分之一。如果设置了-Xmn的情况下，该参数是不需要在设置的。 
* -XX:PermSize 设置持久代初始值，默认是物理内存的六十四分之一 
* -XX:MaxPermSize 设置持久代最大值，默认是物理内存的四分之一 
* -XX:MaxTenuringThreshold 新生代中对象存活次数，默认15。(若对象在eden区，经历一次MinorGC后还活着，则被移动到Survior区，年龄加1。以后，对象每次经历MinorGC，年龄都加1。达到阀值，则移入老年代) 
* -XX:SurvivorRatio Eden区与Subrvivor区大小的比值，如果设置为8，两个Subrvivor区与一个Eden区的比值为2:8，一个Survivor区占整个新生代的十分之一 
* -XX:+UseFastAccessorMethods 原始类型快速优化 
* -XX:+AggressiveOpts 编译速度加快 
* -XX:PretenureSizeThreshold 对象超过多大值时直接在老年代中分配
### 最佳实践
1. Xmn用于设置新生代的大小。过小会增加Minor GC频率，过大会减小老年代的大小。一般设为整个堆空间的1/4或1/3. 
2. XX:SurvivorRatio用于设置新生代中survivor空间(from/to)和eden空间的大小比例；XX:TargetSurvivorRatio表示，当经历Minor GC后，survivor空间占有量(百分比)超过它的时候，就会压缩进入老年代(当然，如果survivor空间不够，则直接进入老年代)。默认值为50%。 
3. 为了性能考虑，一开始尽量将新生代对象留在新生代，避免新生的大对象直接进入老年代。因为新生对象大部分都是短期的，这就造成了老年代的内存浪费，并且回收代价也高(Full GC发生在老年代和方法区Perm) 
4. 当Xms=Xmx，可以使得堆相对稳定，避免不停震荡 
5. 一般来说，MaxPermSize设为64MB可以满足绝大多数的应用了。若依然出现方法区溢出，则可以设为128MB。若128MB还不能满足需求，那么就应该考虑程序优化了，减少动态类的产生。
