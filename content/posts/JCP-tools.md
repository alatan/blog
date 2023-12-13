---  
title: "线程协作工具与并发容器"  
date: 2018-04-09
weight: 70  
draft: false  
keywords: [""]  
description: "线程协作工具与并发容器"  
tags: ["Java并发编程"]
categories: ["Java并发编程"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

## 线程协作工具类
线程协作工具类，控制线程协作的工具类，帮助程序员让线程之间的协作变得更加简单
### CountDownLatch计数门闩：
* 倒数结束之前，一直处于等待状态，直到数到0，等待线程才继续工作。
* 场景：购物拼团、分布式锁
* 方法：
    * new CountDownLatch(int count)
    * await()：调用此方法的线程会阻塞，支持多个线程调用，当计数为0，则唤醒线程
    * countdown()：其他线程调用此方法，计数减1

### Semaphore信号量：
* 限制和管理数量有限的资源的使用
* 场景：Hystrix、Sentinel限流
* 方法：
    * new Semaphore ((int permits) 可以创建公平的非公平的策略
    * acquire()：获取许可证，获取许可证，要么获取成功，信号量减1，要么阻塞等待唤醒
    * release()：释放许可证，信号量加1，然后唤醒等待的线程

### CyclicBarrier循环栅栏：
* 线程会等待，直到线程到了事先规定的数目，然后触发执行条件进行下一步动作
* 场景：并行计算
* 方法：
    * new CyclicBarrier(int parties, Runnable barrierAction)参数1集结线程数，参数2凑齐之后执行的任务
    * await()：阻塞当前线程，待凑齐线程数量之后继续执行

### Condition接口：
* 控制线程的“等待”和“唤醒”
* 方法：
    * await()：阻塞线程
    * signal()：唤醒被阻塞的线程
    * signalAll()会唤起所有正在等待的线程。
* 注意：
    * 调用await()方法时必须持有锁，否则会抛出异常
    * Condition和Object#await/notify方法用法一样，两者await方法都会释放锁

### 对比
![](/images/current/juc/JCP-Tools.png "协作工具对比")

## 并发容器
什么是并发容器？
针对多线程并发访问来进行设计的集合，称为并发容器
* JDK1.5之前，JDK提供了线程安全的集合都是同步容器，线程安全，只能串行执行，性能很差。
* JDK1.5之后，JUC并发包提供了很多并发容器，优化性能，替代同步容器

### CopyOnWriteArrayList
* CopyOnWriteArrayList底层数组实现，使用复制副本进行有锁写操作，适合读多写少，允许短暂的数据不一致的场景。
* CopyOnWrite思想：平时查询时，不加锁，更新时从原来的数据copy副本，然后修改副本，最后把原数据替换为副本。修改时，不阻塞读操作，读到的是旧数据。
* 优缺点
    * 优点：对于读多写少的场景， CopyOnWrite这种无锁操作性能更好，相比于其它同步容器
    * 缺点：①数据一致性问题，②内存占用问题及导致更多的GC次数

### ConcurrentHashMap
[ConcurrentHashMap详解](https://mp.weixin.qq.com/s?__biz=MzIwNTI2ODY5OA==&mid=2649938471&idx=1&sn=2964df2adc4feaf87c11b4915b9a018e "ConcurrentHashMap详解")

## 并发队列
队列是线程协作的利器，通过队列可以很容易的实现数据共享，并且解决上下游处理速度不匹配的问题，典型的生产者消费者模式
### 什么是阻塞队列 
* 带阻塞能力的队列，阻塞队列一端是给生产者put数据使用，另一端给消费者take数据使用
* 阻塞队列是线程安全的，生产者和消费者都可以是多线程
* take方法：获取并移除头元素，如果队列无数据，则阻塞
* put方法：插入元素，如果队列已满，则阻塞
* 阻塞队列又分为有界和无界队列，无界队列不是无限队列，最大值Integer.MAX_VALU
### 常用阻塞队列
1. ArrayBlockingQueue 基于数组实现的有界阻塞队列
2. LinkedBlockingQueue 基于链表实现的无界阻塞队列
3. SynchronousQueue不存储元素的阻塞队列
4. PriorityBlockingQueue 支持按优先级排序的无界阻塞队列
5. DelayQueue优先级队列实现的双向无界阻塞队列
6. LinkedTransferQueue基于链表实现的无界阻塞队列
7. LinkedBlockingDeque基于链表实现的双向无界阻塞队列

## ThreadLocal
*ThreadLocal是线程本地变量类，在多线程并执行过程中，将变量存储在ThreadLocal中，每个线程中都有独立的变量，因此不会出现线程安全问题。*

ThreadLocal 是解决线程安全问题一个较好方案，它通过为每个线程提供一个独立的本地值，去解决并发访问的冲突问题。很多情况下，使用 ThreadLocal 比直接使用同步机制（如 synchronized）解决线程安全问题更简单，更方便，且结果程序拥有更高的并发性。

* ThreadLocal在Spring中作用巨大，在管理Request作用域中的Bean、事务、任务调度、AOP等模块都有它。
* Spring中绝大部分Bean都可以声明成Singleton作用域，采用ThreadLocal进行封装，因此有状态的Bean就能够以singleton的方式在多线程中正常工作了。

###  ThreadLocal底层原理
JDK1.8之前：ThreadLocal是Map所有线程拥有同一个，Key为thread，Value为具体值
JDK1.8：ThreadLocal依旧是Map，但一个线程一个ThreadLocalMap，key为ThreadLocal，Value为具体值
主要变化：①ThreadLocalMap的拥有者，②Key
#### JDK1.8中Thread、ThreadLocal、ThreadLocalMap的关系？
➢ Thread --> ThreadLocalMap --> Entry(ThreadLocalN, LocalValueN)*n
###  Entry的Key为什么需要使用弱引用
![](/images/currentJCP-threadLocalEntry.png "Entry的Key为什么需要使用弱引用")