---  
title: "Java内存模型(JMM)"  
date: 2018-04-02
weight: 70  
draft: false  
keywords: [""]  
description: "Java并发编程-内存模型"  
tags: ["Java并发编程"]
categories: ["Java并发编程"]  
author: "默哥"  

lightgallery: true

toc:
  auto: 
---  

## 并发编程模型的分类
在并发编程中，我们需要处理两个关键问题：线程之间如何通信及线程之间如何同步（这里的线程是指并发执行的活动实体）。通信是指线程之间以何种机制来交换信息。在命令式编程中，线程之间的通信机制有两种：共享内存和消息传递。 

在共享内存的并发模型里，线程之间共享程序的公共状态，线程之间通过写 - 读内存中的公共状态来隐式进行通信。在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显式进行通信。 

同步是指程序用于控制不同线程之间操作发生相对顺序的机制。在共享内存并发模型里，同步是显式进行的。程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的。 

**Java的并发采用的是共享内存模型，Java 线程之间的通信总是隐式进行，整个通信过程对程序员完全透明。如果编写多线程程序的 Java 程序员不理解隐式进行的线程之间通信的工作机制，很可能会遇到各种奇怪的内存可见性问题。**

## CPU和缓存一致性
* 在多核 CPU 中每个核都有自己的缓存，同一个数据的缓存与内存可能不一致
* **为什么需要CPU缓存**？随着CPU技术发展，CPU执行速度和内存读取速度差距越来越大，导致CPU每次操作内存都要耗费很多等待时间。为了解决这个问题，在CPU和物理内存上新增高速缓存。
* 程序在运行过程中会将运算所需数据**从主内存复制到CPU高速缓存**，当CPU计算直接操作高速缓存数据，**运算结束将结果刷回主内存**。

## Java内存模型（Java Memory Model）
* Java为了保证满足原子性、可见性及有序性，诞生了一个重要的规范JSR133，Java内存模型简称JMM
* JMM定义了共享内存系统中**多线程应用读写操作行为的规范**
* JMM规范定义的规则，规范了内存的读写操作，从而保证指令执行的正确性
* **JMM规范解决了CPU多级缓存、处理器优化、指令重排等导致的内存访问问题**
* Java实现了JMM规范因此有了Synchronized、Volatile、锁等概念
* JMM的实现屏蔽各种硬件和操作系统的差异，在各种平台下对内存的访问都能保证效果一致

### JMM内存模型抽象结构
**JMM 通过控制主内存与每个线程的本地内存之间的交互，来为 java 程序员提供内存可见性保证。**

Java内存模型的抽象示意图如下：
![](/images/jvm/java-jmm.png "Java内存模型的抽象示意图")

* JMM定义共享变量何时写入，何时对另一个线程可见
* 线程之间的共享变量存储在主内存
* 每个线程都有一个私有的本地内存，本地内存存储共享变量的副本
* 本地内存是抽象的，不真实存在，涵盖：缓存，写缓冲区，寄存器等

#### JMM线程操作内存基本规则
* 线程操作共享变量必须在本地内存中，不能直接操作主内存的
* 线程间无法直接访问对方的共享变量，需经过主内存传递

### happens-before规则
在JMM中，使用happens-before规则来约束编译器的优化行为，允许编译期优化，但需要遵守一定的Happens-Before规则。如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须要存在happens-before的关系！
#### 与程序员密切相关的 happens-before 规则如下：
* 程序顺序规则：一个线程中的每个操作，happens- before 于该线程中的任意后续操作。 
* 监视器锁规则：对一个监视器锁的解锁，happens- before 于随后对这个监视器锁的加锁。 
* volatile 变量规则：对一个 volatile 域的写，happens- before 于任意后续对这个 volatile 域的读。 
* 传递性：如果 A happens- before B，且 B happens- before C，那么 A happens- before C。
#### happens-before的实现
*注意：对于程序员来说，理解以上happens-before规则即可，JMM设计happens-before的目标就是屏蔽编译器和处理器重排序规则的复杂性*
1. 处理器重排序规则
2. 编译器重排序规则

#### Happens-Before 规则 
1. 单一线程原则（在一个线程内，在程序前面的操作先行发生于后面的操作。）
2. 管程锁定规则（一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。）
3. volatile 变量规则（对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。）
4. 线程启动规则（Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。）
5. 线程加入规则（Thread 对象的结束先行发生于 join() 方法返回。）
6. 线程中断规则（对线程 interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过interrupted()方法检测到是否有中断发生。）
7. 对象终结规则 一个对象的初始化完成(构造函数执行结束)先行发生于它的 finalize() 方法的开始。
8. 传递性（如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。）

### 重排序
在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：
* 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。 
* 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。 
* 内存系统的重排序。由于处理器使用缓存和读 / 写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

从 java 源代码到最终实际执行的指令序列，会分别经历下面三种重排序：
![](/images/jvm/jmm-order.png "JMM指令重排序")

上述的 1 属于编译器重排序，2 和 3 属于处理器重排序。这些重排序都可能会导致多线程程序出现内存可见性问题。对于编译器，JMM 的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）。对于处理器重排序，JMM 的处理器重排序规则会要求 java 编译器在生成指令序列时，插入特定类型的内存屏障（memory barriers，intel 称之为 memory fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序（不是所有的处理器重排序都要禁止）。

**JMM属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。**

### 处理器重排序与内存屏障指令
为了保证内存可见性，java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。

StoreLoad Barriers 是一个“全能型”的屏障，它同时具有其他三个屏障的效果。现代的多处理器大都支持该屏障（其他类型的屏障不一定被所有处理器支持）。执行该屏障开销会很昂贵，因为当前处理器通常要把写缓冲区中的数据全部刷新到内存中（buffer fully flush）。 