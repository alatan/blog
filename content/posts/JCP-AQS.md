---  
title: "AQS分析"  
date: 2018-04-07
weight: 70  
draft: false  
keywords: [""]  
description: "AQS分析"  
tags: ["Java并发编程"]
categories: ["Java并发编程"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

## 引子
![](/images/current/aqs/AQS-class.png "类图外层")
重入锁ReentrantLock类关系图，它是实现了Lock接口的类。NonfairSync和FairSync都继承自抽象类Sync，在ReentrantLock中有非公平锁NonfairSync和公平锁FairSync的实现。在重入锁ReentrantLock类关系图中，我们可以看到NonfairSync和FairSync都继承自抽象类Sync，而Sync类继承自抽象类AbstractQueuedSynchronizer（简称AQS）。如果我们看过JUC的源代码，会发现不仅重入锁用到了AQS， JUC 中绝大部分的同步工具也都是基于AQS构建的

## AQS简介
**AQS（全称AbstractQueuedSynchronizer）即队列同步器**。它是**构建锁**或者其他**同步组件的基础框架**（如ReentrantLock、ReentrantReadWriteLock、Semaphore等）。
* AQS是JUC并发包中的核心基础组件，其本身是一个抽象类。理论上还是利用管程实现的，在AQS中，有一个volatile修饰的state，获取锁的时候，会读写state的值，解锁的时候，也会读写state的值。所以AQS就拥有了volatile的happensbefore规则。加锁与解锁的效果上与synchronized是相同的。

由类图可以看到，AQS是一个FIFO的双向队列，队列中存储的是thread，其内部通过节点head和tail记录队首
和队尾元素，队列元素的类型为Node。

![](/images/current/aqs/AQS-classAll.png "类图")

## AQS实现原理
AQS是一个同步队列，内部使用一个FIFO的双向链表，管理线程同步时的所有被阻塞线程。双向链表这种数据结构，它的每个数据节点中都有两个指针，分别指向直接后继节点和直接前驱节点。所以，从双向链表中的任意一个节点开始，都可以很方便地访问它的前驱节点和后继节点。

我们看下面的AQS的数据结构，AQS有两个节点head，tail分别是头节点和尾节点指针，默认为null。AQS中的内部静态类Node为链表节点，AQS会在线程获取锁失败后，线程会被阻塞并被封装成Node加入到AQS队列中；**当获取锁的线程释放锁后，会从AQS队列中的唤醒一个线程（节点）**。

![](/images/current/aqs/AQS-dataStruct.png "AQS的数据结构")

### 场景01-线程抢夺锁失败时，AQS队列的变化【加锁】
1. AQS的head、tail分别代表同步队列头节点和尾节点指针默认为null
2. 当第一个线程抢夺锁失败，同步队列会先初始化，随后线程会被封装成Node节点追加到AQS队列中。
	* 假设：当前独占锁的的线程为ThreadA，抢占锁失败的线程为ThreadB。
	* 2.1 同步队列初始化，首先在队列中添加Node，thread=null
	* 2.2 将ThreadB封装成为Node，追加到AQS队列
3. 当下一个线程抢夺锁失败时，继续重复上面步骤。假设：ThreadC抢占线程失败

![](/images/current/aqs/AQS-addLock.png "AQS队列加锁")

### 场景02-线程被唤醒时，AQS队列的变化【解锁】
1. ReentrantLock唤醒阻塞线程时，会按照FIFO的原则从AQS中head头部开始唤醒首个节点中线程。
2. head节点表示当前获取锁成功的线程ThreadA节点。
3. 当ThreadA释放锁时，它会唤醒后继节点线程ThreadB，ThreadB开始尝试获得锁，如果ThreadB获得锁成功，会将自己设置为AQS的头节点。ThreadB获取锁成功后，AQS变化如下：

![](/images/current/aqs/AQS-unLock.png "AQS队列解锁")

## 锁源码分析
### 锁的获取
![](/images/current/aqs/AQS-sourceGetLock.png "锁的获取")

### 锁的释放
![](/images/current/aqs/AQS-sourceRelaseLock.png "锁的释放")

### 公平锁和非公平锁源码实现区别
公平锁和非公平锁在获取锁和释放锁时有什么区别呢？
* 非公平锁与非公平锁释放锁是没有差异，释放锁时调用方法都是AQS的方法。
* 非公平锁与非公平锁获取锁的差异
	* 我们可以看到上面在公平锁中，线程获得锁的顺序按照请求锁的顺序，按照先来后到的规则获取锁。如果线程竞争公平锁失败后，都会到AQS同步队列队尾排队，将自己阻塞等待锁的使用资格，锁被释放后，会从队首开始查找可以获得锁的线程并唤醒。
	* 而非公平锁，允许新线程请求锁时，可以插队，新线程先尝试获取锁，如果获取锁失败，才会AQS同步队列队尾排队。

## 读写锁ReentrantReadWriteLock
可重入锁ReentrantLock是互斥锁，互斥锁在同一时刻仅有一个线程可以进行访问，但是在大多数场景下，大部分时间都是提供读服务，而写服务占有的时间较少。然而读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读势必会导致性能降低，所以就出现了读写锁。

**读写锁维护着一对锁，一个读锁和一个写锁**。通过分离读锁和写锁，使得并发性比一般的互斥锁有了较大的提升：在同一时间可以允许多个读线程同时访问，但是在写线程访问时，所有读线程和写线程都会被阻塞。

**读写锁的主要特性**：
* 公平性：支持公平性和非公平性。
* 重入性：支持重入。读写锁最多支持65535个递归写入锁和65535个递归读取锁。
* 锁降级：写锁能够降级成为读锁，但读锁不能升级为写锁。遵循获取写锁、获取读锁在释放写锁的次序