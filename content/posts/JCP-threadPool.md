---
title: "线程池"  
date: 2018-04-10
weight: 70  
draft: false  
keywords: [""]  
description: "线程池"  
tags: ["Java并发编程"]
categories: ["Java并发编程"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---
## 线程池
线程池（ThreadPool） 是一种基于池化思想管理线程的工具，看过new Thread源码之后我们发现，频繁创建线程销毁线程的开销很大，会降低系统整体性能。线程池维护多个线程，等待监督和管理分配可并发执行的任务。

### 优点
* 降低资源消耗：通过线程池复用线程，降低创建线程和释放线程的损耗
* 提高响应速度：任务到达时，无需等待即刻运行
* 提高线程的可管理性：使用线程池可以进行统一的分片、调优和监控线程
* 提供可扩展性：线程池具备可扩展性，研发人员可以向其中增加各种功能，比如延时，定时，监控等
### 适用场景
* 连接池：预先申请数据库连接，提升申请连接的速度，降低系统的开销
* 快速响应用户请求：服务器接受到大量请求时，使用线程池是很适合的，它可以大大减少线程的创
* 建和销毁的次数，提高服务器的工作效率。
* 在实际开发中，如果需要创建5个以上的线程，就可以用线程池来管理。

## 线程池参数
* corePoolSize：核心线程数，可以理解为空闲线程数，即便线程空闲（Idle），也不会回收
* maxPoolSize ：最大线程数，线程池可以容纳线程的上限
* keepAliveTime ：线程保持存活的时间，超过核心线程数的线程存活空闲时间超过keepAliveTime后就会被回收
* workQueue ：工作队列，直接交换队列SynchronousQueue，无界队列LinkedBlockingQueue ，有界队列ArrayBlockingQueue
* threadFactory：线程工厂，用来创建线程的工厂，线程都是出自于此工厂
* Handler：线程无法接收任务时的拒绝策略
## 线程池原理
![](/images/current/JCP-threadPool.png "线程池原理")

* 提交任务，如果线程数小于corePoolSize即使其他线程处于空闲状态，也会创建一个新线程来运行任务
* 如果线程数大于corePoolSize，但少于maxPoolSize，将任务放入工作队列
* 如果队列已满，并且线程数小于maxPoolSize，则创建一个新线程来运行任务。
* 如果队列已满，并且线程数大于或等于maxPoolSize，则拒绝该任务。
## 自动创建线程
* newFixedThreadPool：固定数量线程池，无界任务阻塞队列
* newSingleThreadExecutor ：一个线程的线程池，无界任务阻塞队列
* newCachedThreadPool ：可缓存线程的无界线程池，可以自动回收多余线程
* newScheduledThreadPool ：定时任务线程池
## 手动创建线程
根据不同的业务场景，自己设置线程池的参数、线程名、任务被拒绝后如何记录日志等
### 如何设置线程池大小？
* CPU密集型：线程数量不能太多，可以设置为与相当于CPU核数
* IO密集型：IO密集型CPU使用率不高，可以设置的线程数量多一些，可以设置为CPU核心数的2倍
### 拒绝策略
* 拒绝时机：①最大线程和工作队列有限且已经饱和，②Executor关闭时
* 抛异常策略：AbortPolicy，说明任务没有提交成功
* 不做处理策略：DiscardPolicy，默默丢弃任务，不做处理
* 丢弃老任务策略：DiscardOldestPolicy，将队列中存在最久的任务给丢弃
* 自产自销策略：CallerRunsPolicy，那个线程提交任务就由那个线程负责运行

## Future与FutureTask
FutureTask叫未来任务，可以将一个复杂的任务剔除出去，交给另一个线程来完。它是Future的实现类
### Future用法01-用线程池submit方法提交任务，返回值Future任务结果
* 用线程池提交任务，线程池会立即返回一个空的Future容器
* 当线程的任务执行完成，线程池会将该任务执行结果填入Future中
* 此时就可以从Future获取执行结果
### Future用法02-用FutureTask来封装任务，获取Future任务的结果
* 用FutureTask包装任务，FutureTask是Future和Runnable接口的实现类
* 可以使用new Thread().start()或线程池执行FutureTask
* 任务执行完成，可以从FutureTask中获取执行结果