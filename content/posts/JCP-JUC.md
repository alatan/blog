---  
title: "JUC-并发编程利器"  
date: 2018-04-05
weight: 70  
draft: false  
keywords: [""]  
description: "JUC并发编程利器"  
tags: ["Java并发编程"]
categories: ["Java并发编程"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

## JUC简介
从JDK1.5起，Java API 中提供了java.util.concurrent（简称JUC）包，在此包中定义了并发编程中很常用的工具，比如：线程池、阻塞队列、同步器、原子类等等。JUC是 JSR 166 标准规范的一个实现，JSR166 以及 JUC 包的作者是同一个人 Doug Lea 。

![](/images/current/juc/juc-overview.png "JUC概览")
![](/images/current/juc/juc-overview-base.png "JUC层次")

## Atomic包
从JDK 1.5开始提供了java.util.concurrent.atomic包（以下简称Atomic包），这个包中的原子操作类提供了一种用法简单、性能高效、线程安全地更新一个变量的方式。可以解决volatile原子性操作变量的问题。
### Atomic包里的类
* 基本类型：AtomicInteger整形原子类…
* 引用类型：AtomicReference引用类型原子类…
* 数组类型： AtomicIntegerArray整形数组原子类…
* 对象属性修改类型：AtomicIntegerFieldUpdater原子更新整形字段的更新器…
* JDK1.8新增：DoubleAdder双浮点型原子类、LongAdder长整型原子类…
虽然原子类很多，但原理几乎都差不多，其核心是采用CAS进行原子操作

## CAS
CAS即compare and swap（比较再替换），同步组件中大量使用CAS技术实现了Java多线程的并发操作。整个AQS、Atomic原子类底层操作，都可以看见CAS。甚至ConcurrentHashMap在1.8的版本中也调整为了CAS+Synchronized。可以说CAS是整个JUC的基石。
**CAS本质是一条CPU的原子指令，可以保证共享变量修改的原子性。**。

### CAS缺陷
CAS虽然高效地解决了原子操作，但是还是存在一些缺陷的，主要表现在三个地方：**循环时间太长、只能保证一个共享变量原子操作、ABA问题**。
* 循环时间太长：如果CAS一直不成功呢？如果自旋CAS长时间地不成功，则会给CPU带来非常大的开销。
    原子类AtomicInteger#getAndIncrement()的方法
* 只能保证一个共享变量原子操作：看了CAS的实现就知道这只能针对一个共享变量，如果是多个共享变量就只能使用锁了。
* ABA问题：CAS需要检查操作值有没有发生改变，如果没有发生改变则更新。但是存在这样一种情况：如果一个值原来是A，变成了B，然后又变成了A，那么在CAS检查的时候会发现没有改变，但是实质上它已经发生了改变，这就是所谓的ABA问题。**对于ABA问题其解决方案是加上版本号，即在每个变量绑定一个版本号，每次改变时加1，即A —> B —> A，变成1A —> 2B —> 3A**。

### 锁
* LockSupport
* ReentrantLock
* ReentrantReadWriteLock

### 并发集合


### 线程池
* FutureTask
* ThreadPoolExecutor
* ScheduledThreadPoolExecutor
* Fork/Join

### 工具类
* CountDownLatch
* CyclicBarrier
* Semaphore
* Phaser
* Exchanger
* ThreadLocal