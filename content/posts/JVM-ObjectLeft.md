---  
title: "JVM-一个对象的一生（出生、死亡与内涵）"  
date: 2018-05-04
weight: 70  
draft: false  
keywords: [""]  
description: "JVM-一个对象的一生（出生、死亡与内涵）"  
tags: ["JVM"]
categories: ["JVM"]
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

![](/images/jvm/jvm-objectCreate.png)

## 内存分配的方法有两种：不同垃圾收集器不一样
* 指针碰撞(Bump the Pointer)
* 空闲列表(Free List)

| 分配方法    | 说明 | 收集器   |
| :---   | :--- | :---  |
| 指针碰撞(Bump thePointer)    | 内存地址是连续的（新生代） |  Serial 和 ParNew 收集器    |
| 比空闲列表(Free List)   | 内存地址不连续（老年代）  |  数字CMS 收集器和 Mark-Sweep 收集器 |

![](/images/jvm/jvm-objectCreateMethod.png)
### 内存分配安全问题
*虚拟机给A线程分配内存的过程中，指针未修改，此时B线程同时使用了该内存，有问题*

处理方案
* CAS乐观锁：JVM虚拟机采用CAS失败重试的方式保证更新操作的原子性
* TLAB （Thread Local Allocation Buffer） 本地线程分配缓存，预分配

**分配主流程**：首先从TLAB里面分配，如果分配不到，再使用CAS从堆里面划分

## 对象内存分配流程
![](/images/jvm/jvm-ObjectCreateFlow.png)

为对象分配内存是一件非常严谨和复杂的任务，JVM 的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法和内存回收算法密切相关，所以还需要考虑 GC 执行完内存回收后是否会在内存空间中产生内存碎片。 
1. new 的对象先放在伊甸园区，此区有大小限制 
2. 当伊甸园的空间填满时，程序又需要创建对象，JVM 的垃圾回收器将对伊甸园区进行垃圾回收（Minor GC），将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区 
3. 然后将伊甸园中的剩余对象移动到幸存者 0 区 
4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者 0 区，如果没有回收，就会放到幸存者 1 区 
5. 如果再次经历垃圾回收，此时会重新放回幸存者 0 区，接着再去幸存者 1 区 
6. 什么时候才会去养老区呢？ 默认是 15 次回收标记 
7. 在养老区，相对悠闲。当养老区内存不足时，再次触发 Major GC，进行养老区的内存清理 
8. 若养老区执行了 Major GC  之后发现依然无法进行对象的保存，就会产生 OOM 异常


### 对象内存分配
#### 新生代：新对象大多数都默认进入新生代的Eden区
#### 进入老年代的条件：四种情况
* 存活年龄太大，默认超过15次【-XX:MaxTenuringThreshold】
* 动态年龄判断：MinorGC之后，发现Survivor区中的一批对象的总大小大于了这块Survivor区的50%，那么就会将此时大于等于这批对象年龄最大值的所有对象，直接进入老年代。
  * 举个栗子：Survivor区中有一批对象，年龄分别为年龄1+年龄2+年龄n的多个对象，对象总和大小超过了Survivor区域的50%，此时就会把年龄n及以上的对象都放入老年代。
  * 为什么会这样？希望那些可能是长期存活的对象，尽早进入老年代。
  * -XX:TargetSurvivorRatio可以指定
* 大对象直接进入老年代：**前提是Serial和ParNew收集器**
   * 举个栗子：字符串或数组
  * -XX:PretenureSizeThreshold 一般设置为1M
  * 为什么会这样？为了避免大对象分配内存时的复制操作降低效率。避免了Eden和Survivor区的复制
* MinorGC后，存活对象太多无法放入Survivor

#### 空间担保机制
当新生代无法分配内存的时候，我们想把新生代的**老对象**转移到老年代，然后把**新对象**放入腾空的新生代。此种机制我们称之为**内存担保**。
* MinorGC前，判断老年代可用内存是否小于新时代对象全部对象大小，如果小于则继续判断
* 判断老年代可用内存大小是否小于之前每次MinorGC后进入老年代的对象平均大小
  * 如果是，则会进行一次FullGC，判断是否放得下，放不下OOM
  * 如果否，则会进行一些MinorGC：
    * MinorGC后，剩余存活对象小于Survivor区大小，直接进入Survivor区
    * MinorGC后，剩余存活对象大于Survivor区大小，但是小于老年代可用内存，直接进入老年代
    * MinorGC后，剩余存活对象大于Survivor区大小，也大于老年代可用内存，进行FullGC
    * FullGC之后，任然没有足够内存存放MinorGC的剩余对象，就会OOM

##### 老年代的担保示意图
![](/images/jvm/jvm-ObjectCreateDB.png)

#### 小结
1. 当Eden区存储不下新分配的对象时，会触发minorGC
2. GC之后，还存活的对象，按照正常逻辑，需要存入到Survivor区。
3. 当无法存入到幸存区时，此时会触发担保机制
4. 发生内存担保时，需要将Eden区GC之后还存活的对象放入老年代。后来的新对象或者数组放入Eden区。

## 对象内存布局
![](/images/jvm/jvm-ObjectMemoryBJ.png)

![](/images/jvm/jvm-ObjectCreateDB2.png)

### 对象里的三个区
#### 对象头
##### 标记字段：存储对象运行时自身数据
* 默认：对象Hashcode，GC分代年龄，锁状态
* 存储数据结构并不是固定的
* 这部分在 64 位操作系统下占 8 字节，32 位操作系统下占 4 字节。
##### 类型指针：对象指向类元数据的指针
* 对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪一个类的实例。
* 开启指针压缩占4字节，不开启8字节，默认是开启的
##### 数组长度
这部分只有是数组对象才有，若是是非数组对象就没这部分。这部分占 4 字节。

对象头信息是与对象自身定义的数据无关的额外存储成本。考虑到虚拟机的空间效率，Mark Word被设计成一个非固定的数据结构，以便在极小的空间内，尽量多的存储数据，它会根据对象的状态复用自己的存储空间，也就是说，Mark Word会随着程序的运行发生变化，变化状态如下（JDK1.8）。
![](/images/jvm/jvm-ObjectCreateDXT.png)

#### 实例数据
用于存储对象中的各类类型的字段信息（包括从父类继承来的）
#### 对齐填充
* Java 对象的大小默认是按照 8 字节对齐，也就是说 Java 对象的大小必须是 8 字节的倍数。若是算到最后不够 8 字节的话，那么就会进行对齐填充。

* 那么为何非要进行 8 字节对齐呢？这样岂不是浪费了空间资源？

* 其实不然，由于 CPU 进行内存访问时，一次寻址的指针大小是 8 字节，正好也是 L1 缓存行的大小。如果不进行内存对齐，则可能出现跨缓存行的情况，这叫做 缓存行污染。

## 如何访问一个对象
### 句柄：稳定，对象被移动只要修改句柄中的地址
![](/images/jvm/jvm-ObjectFwdx.png)

###  直接指针：访问速度快，节省了一次指针定位的开销
![](/images/jvm/jvm-ObjectFwdxZz.png)