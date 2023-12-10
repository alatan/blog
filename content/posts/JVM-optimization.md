---  
title: "JVM调优"  
date: 2018-05-07
weight: 70  
draft: false  
keywords: [""]  
description: "JVM调优"  
tags: ["JVM"]
categories: ["JVM"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

## 调优思想
### 为什么JVM调优
降本增效：调优的最终目的都是为了应用程序使用最小的硬件消耗来承载更大的吞吐量。JVM调优主要是针对垃圾收集器的收集性能进行优化令运行在虚拟机上的应用，能够使用更少的内存（Footprint），及更低的延迟（Latency），获取更大的吞吐量（Throughput）。

#### 调优目标(不同应用场景的JVM调优量化目标是不一样的，这里的目标只一个参照模板)
* 堆内存使用率 <= 70%;
* 老年代内存使用率<= 70%;
* avg pause <= 1秒;
* Full GC 次数0 或 avg pause interval >= 24小时 ;

### 什么时候JVM调优
1. 系统吞吐量下降与响应延迟（P99）；
2. Heap内存（老年代）持续上涨至出现OOM；
3. Full GC 次数频繁；
4. GC 停顿过长（超过1秒）；
5. 应用出现OutOfMemory 等内存异常；
6. 应用中有使用本地缓存且占用大量内存空间；

### 调什么
内存分配 + 垃圾回收！
1. 合理使用堆内存
2. GC高效回收占用的内存的垃圾对象
3. GC高效释放掉内存空间

### 调优的原则和目标如何确定？
* 优先原则：产品、架构、代码、数据库优先，JVM是不得已的最后的手段(大多数的Java应用不需要进行JVM优化)
* 观测性原则：发现问题解决问题，没问题不创造问题。

## 调优主要步骤
![](/images/jvm/jvm-ty-bz.png)

* 第一步：监控分析GC日志
* 第二步：判断JVM问题：
    * 如果各项参数设置合理，系统没有超时日志出现，GC频率不高，GC耗时不高，那么没有必要进行GC优化
    * 如果GC时间超过1秒，或者频繁GC，则必须优化。
* 第三步：确定调优目标
* 第四步：调整参数
    * 调优一般是从满足程序的内存使用需求开始，之后是时间延迟需求，最后才是吞吐量要求，要基于这个步骤来不断优化，每一个步骤都是进行下一步的基础，不可逆行之。
* 第五步：对比调优前后差距
* 第六步：重复：1、2、3、4、5步骤
    * 找到最佳JVM参数设置
* 第七步：应用JVM参数到应用服务器：
    * 找到最合适的参数，将这些参数灰度发布一部分机器，观察一段时间。
    * 如果，观察结果可以验证方案的价值，则进行全量发布！

##  GC日志
### 参数配置
JVM调优典型参数设置：
* -Xms堆内存最小值
* -Xmx堆内存最大值
* -Xmn新生代内存的最大值
* -Xss每个线程的栈内存

建议：在开发测试环境可以用Xms和Xmx设置最小值最大值，但是在线上生产环境，Xms和Xmx设置的值相同防止抖动；

### 如果想要确定JVM性能问题瓶颈，需要分析GC日志
1. -XX:+PrintGCDetails 开启GC日志创建更详细的GC日志，默认关闭
2. -XX:+PrintGCTimeStamps，-XX:+PrintGCDateStamps 开启GC时间提示，
    * 开启时间便于我们更精确地判断几次GC操作之间的时两个参数的区别
    * 时间戳是相对于0（依据JVM启动的时间）的值，而日期戳（date stamp）是实际的日期字符串
    * 由于日期戳需要进行格式化，所以它的效率可能会受轻微的影响，不过这种操作并不频繁，它造成的影响也很难被我们感知。
3. -XX:+PrintHeapAtGC 打印堆的GC日志
4. -Xloggc:./logs/gc.log 指定GC日志路径

```yml
# 配置GC日志输出
JAVA_OPT="${JAVA_OPT} -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-default.log "
```

###  GC日志解读
#### Young GC 日志含义
```yml
2021-05-18T14:31:11.340+0800: 2.340: [GC (Allocation Failure) [PSYoungGen: 896512K->41519K(1045504K)]
896512K-41543K(3435008K), 0.0931965 secs] [Times: user=0.14 sys=0.02, real=0.10 secs]
# 第一部分
2021-05-18T14:31:11.340+0800: 2.340: [GC (Allocation Failure)
GC事件开始时间，+0800代表中国所在的东区：2021-05-18T14:31:11.340+0800
GC事件开始时间相对于JVM开始启动的间隔秒数：2.340
区分YoungGC还是FullGC的标志，GC代表YoungGC
触发GC的原因： (Allocation Failure)
# 第二部分
[PSYoungGen: 896512K->41519K(1045504K)] 896512K-41543K(3435008K), 0.0931965 secs]
垃圾收集器的名称：PSYoungGen
在垃圾收集之前 和 之后新时代的使用量：896512K->41519K
新生代总空间大小：(1045504K)
在垃圾收集之前和之后整个堆内存使用量：896512K-41543K
堆总空间大小：(3435008K)
GC持续时间：0.0931965 secs
# 第三部分
[Times: user=0.14 sys=0.02, real=0.10 secs]
GC线程消耗的CPU时间：user=0.14
GC过程中操作系统调用和系统等待事件所消耗的时间：sys=0.02
应用程序暂停的时间：0.10 secs
```

#### FullGC 日志含义
```yml
2021-05-19T14:46:07.367+0800: 1.562: [Full GC (Metadata GC Threshold)[PSYoungGen: 18640K-
>0K(1835008K)] [ParOldGen: 16K->18327K(1538048K)] 18656K->18327K(3373056K), [Metaspace: 20401K-
>20398K(1069056K)], 0.0624559 secs] [Times: user=0.19 sys=0.00, real=0.06 secs]

# 第一部分
2021-05-19T14:46:07.367+0800: 1.562: [Full GC (Metadata GC Threshold)
GC事件开始时间，+0800代表中国所在的东区：
GC事件开始时间相对于JVM开始启动的间隔秒数：2.340
区分YoungGC还是FullGC的标志
触发GC的原因： (Allocation Failure)
# 第二部分
[PSYoungGen: 18640K->0K(1835008K)] [ParOldGen: 16K->18327K(1538048K)] 18656K->18327K(3373056K),
垃圾收集器的名称：PSYoungGen
在垃圾收集之前 和 之后新时代的使用量：18640K->0K
新生代总空间大小：(1835008K)
老年代垃圾收集器名称：ParOldGen
在垃圾收集之前和之后老年代的使用量：16K->18327K
老年代总空间大小：(1538048K)
在垃圾收集之前和之后整个堆内存使用量：18656K->18327K
堆总空间大小：(3373056K)
# 第三部分
[Metaspace: 20401K->20398K(1069056K)], 0.0624559 secs] [Times: user=0.19 sys=0.00, real=0.06 secs]
元空间区域垃圾收集器：Metaspace
在垃圾收集之前和之后元空间的使用量：20401K->20398K
元空间大小：(1069056K)
GC事件持续的时间：0.0624559 secs
GC线程消耗的CPU时间：user=0.19
GC过程中，操作系统调用和系统等待事件所消耗的时间：sys=0.00
应用程序暂停时间：real=0.06 secs
```

### GC日志可视化分析
通过GCEasy导入解析查看


## 优化
### 线程堆栈优化
JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256。根据应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成， 如果栈不是很深， 应该是256k够用了，大的应用建议使用512k。

* 注意：这个选项对性能影响较大，需严格测试确定最终大小。
```yml
# 计算最大线程数的公式：
Number of threads = (MaxProcess内存 - JVM内存 - ReservedOsMemory) / (ThreadStackSize)
系统最大可创建的线程数量 = (机器本身可用内存 - (JVM分配的堆内存+JVM元数据区)) / Xss的值
```
### 垃圾回收器优化
#### 吞吐量优先ps+po
默认使用ps+po 垃圾回收器组合： 并行垃圾回收器组合
```yml
JAVA_OPT="${JAVA_OPT} -Xms256m -Xmx256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -Xss512k"
JAVA_OPT="${JAVA_OPT} -XX:+UseParallelGC -XX:+UseParallelOldGC "
JAVA_OPT="${JAVA_OPT} -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-ps-po.log"
```


#### 响应时间优先parnew+cms
使用cms垃圾回收器，垃圾回收器组合： parNew+CMS, cms垃圾回收器在垃圾标记，垃圾清除的时候，和业务线程交叉执行，尽量减少stw时间，因此这垃圾回收器叫做响应时间优先；
```yml
JAVA_OPT="${JAVA_OPT} -Xms256m -Xmx256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -Xss512k"
JAVA_OPT="${JAVA_OPT} -XX:+UseParNewGC -XX:+UseConcMarkSweepGC "
JAVA_OPT="${JAVA_OPT} -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-parnew-cms.log"
```

#### G1全功能但不全能
配置G1只需要简单三步即可：
1. 第一步，开启G1垃圾收集器
2. 第二步，设置堆的最大内存
3. 第三步，设置最大的停顿时间
```yml
JAVA_OPT="${JAVA_OPT} -Xms256m -Xmx256m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -Xss512k"
JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:MaxGCPauseMillis=100"
JAVA_OPT="${JAVA_OPT} -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC -Xloggc:${BASE_DIR}/logs/gc-g-one.log"
```

对于G1垃圾收集器参数设置建议：
* 设置为100-300之间比较合理，不要设置的太短
* 堆内存小于2GB，不建议使用G1

### 最佳实践
1. Xmn用于设置新生代的大小。过小会增加Minor GC频率，过大会减小老年代的大小。一般设为整个堆空间的1/4或1/3. 
2. XX:SurvivorRatio用于设置新生代中survivor空间(from/to)和eden空间的大小比例；XX:TargetSurvivorRatio表示，当经历Minor GC后，survivor空间占有量(百分比)超过它的时候，就会压缩进入老年代(当然，如果survivor空间不够，则直接进入老年代)。默认值为50%。 
3. 为了性能考虑，一开始尽量将新生代对象留在新生代，避免新生的大对象直接进入老年代。因为新生对象大部分都是短期的，这就造成了老年代的内存浪费，并且回收代价也高(Full GC发生在老年代和方法区Perm) 
4. 当Xms=Xmx，可以使得堆相对稳定，避免不停震荡 
5. 一般来说，MaxPermSize设为64MB可以满足绝大多数的应用了。若依然出现方法区溢出，则可以设为128MB。若128MB还不能满足需求，那么就应该考虑程序优化了，减少动态类的产生。

## JVM调优实战场景
### 内存溢出的定位与分析
* 内存溢出在实际的生产环境中经常会遇到，比如，不断的将数据写入到一个集合中，出现了死循环，读取超大的文件等等，都可能会造成内存溢出。
* 如果出现了内存溢出，首先我们需要定位到发生内存溢出的环节，并且进行分析，是正常还是非正常情况，如果是正常的需求，就应该考虑加大内存的设置，如果是非正常需求，那么就要对代码进行修改，修复这个bug。
* 内存溢出的定位与分析MAT

### 检测死锁
* 使用jstack进行分析（jstack 18487 | grep 'BLOCKED' -A 15 --color）
* 使用Arthas进行分析（thread -b）