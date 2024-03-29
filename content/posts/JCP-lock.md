---  
title: "Java锁"  
date: 2018-04-06
weight: 70  
draft: false  
keywords: [""]  
description: "Java锁"  
tags: ["Java并发编程"]
categories: ["Java并发编程"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

## Java锁简介
Java提供了种类丰富的锁，每种锁因其特性的不同，在适当的场景下能够展现出非常高的效率。
* ReentrantLock重入锁
    它具有与使用 synchronized 相同的一些基本行为和语义，但是它的API功能更强大，重入锁相当于synchronized 的增强版，具有synchronized很多所没有的功能。它是一种**独享锁（互斥锁），可以是公平锁，也可以是非公平的锁**。
* ReentrantReadWriteLock读写锁
    它维护了一对锁，ReadLock读锁和WriteLock写锁。读写锁适合读多写少的场景。基本原则：**读锁可以被多个线程同时持有进行访问，而写锁只能被一个线程持有**。可以这么理解：读写锁是个混合体，它既是一个共享锁，也是一个独享锁。
* StampedLock重入读写锁
    JDK1.8引入的锁类型，是对读写锁ReentrantReadWriteLock的增强版。

## Java锁分类
### 按上锁方式划分
* 隐式锁：synchronized，不需要显示加锁和解锁
* 显式锁：JUC包中提供的锁，需要显示加锁和解锁
### 按特性划分
#### 悲观锁/乐观锁
按照线程在使用共享资源时，要不要锁住同步资源，划分为悲观锁和乐观锁
* 悲观锁：JUC锁，synchronized
* 乐观锁：CAS，关系型数据库的版本号机制
#### 重入锁/不可重入锁
按照同一个线程是否可以重复获取同一把锁，划分为重入锁和不可重入锁
* 重入锁：ReentrantLock、synchronized
* 不可重入锁：不可重入锁，与可重入锁相反，线程获取锁之后不可重复获取锁，重复获取会发生死锁
#### 公平锁/非公平锁
按照多个线程竞争同一锁时需不需要排队，能不能插队，划分为公平锁和非公平锁。
* 公平锁：new ReentrantLock(true)多个线程按照申请锁的顺序获取锁
* 非公平锁：new ReentrantLock(false)多个线程获取锁的顺序不是按照申请锁的顺序(可以插队) synchronized
#### 独享锁/共享锁
按照多个线程能不能同时共享同一个锁，锁被划分为独享锁和共享锁。
* 独享锁：独享锁也叫排他锁，synchronized，ReentrantLock， ReentrantReadWriteLock 的WriteLock写锁
* 共享锁：ReentrantReadWriteLock的ReadLock读锁
#### 自旋锁：
* 实现：CAS、轻量级锁
#### 分段锁：
* 实现：ConcurrentHashMap
* ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。
#### 无锁/偏向锁/轻量级锁/重量级锁
* 这四个锁是synchronized独有的四种状态，级别从低到高依次是：无锁、偏向锁、轻量级锁和重量级锁。
* 它们是JVM为了提高synchronized锁的获取与释放效率而做的优化
* 四种状态会随着竞争的情况逐渐升级，而且是不可逆的过程，即不可降级。

## Synchronized和JUC的锁对比
*Java已经提供了synchronized，为什么还要使用JUC的锁呢？*

Synchronize的缺陷：
* 第一： Synchronized无法控制阻塞时长，阻塞不可中断
    * 使用Synchronized，假如占有锁的线程被长时间阻塞（IO、sleep、join），由于线程阻塞时没法释放锁，会导致大量线程堆积，轻则影响性能，重则服务雪崩
    * **JUC的锁可以解决这两个缺陷**
* 第二：：读多写少的场景中，当多个读线程同时操作共享资源时，读操作和读操作不会对共享资源进行修改，所以读线程和读线程是不需要同步的。如果这时采用synchronized关键字，就会导致一个问题，当多个线程都只是进行读操作时，所有线程都只能同步进行，只能有一个读线程可以进行读操作，其他读线程只能等待锁的释放而无法进行读操作。
    * Synchronized不论是读还是写，均需要同步操作，这种做法并不是最优解
    * **JUC的ReentrantReadWriteLock锁可以解决这个问题**


## 锁优化
### 减少锁持有时间
![](/images/current/JCP-LockTime.png "减少锁持有时间")
### 减少锁粒度
* 将大对象拆分成小对象，增加并行度，降低锁竞争。
* ConcurrentHashMap允许多个线程同时进入
### 锁分离
* 根据功能进行锁分离
* ReadWriteLock在读多写少时，可以提高性能。

### 锁消除
* 锁消除是发生在编译器级别的一种锁优化方式。
* 有时候我们写的代码完全不需要加锁，却执行了加锁操作。

锁消除时指虚拟机即时编译器再运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除。

锁消除的主要判定依据来源于逃逸分析的数据支持。意思就是：JVM会判断在一段程序中的同步明显不会逃逸出去从而被其他线程访问到，那JVM就把它们当作栈上数据对待，认为这些数据时线程独有的，不需要加同步。此时就会进行锁消除。 ​

当然在实际开发中，我们很清楚的知道那些地方时线程独有的，不需要加同步锁，但是在Java API中有很多方法都是加了同步的，那么此时JVM会判断这段代码是否需要加锁。如果数据并不会逃逸，则会进行锁消除。比如如下操作：在操作String类型数据时，由于String是一个不可变类，对字符串的连接操作总是通过生成的新的String对象来进行的。因此Javac编译器会对String连接做自动优化。在JDK 1.5之前会使用StringBuffer对象的连续append()操作，在JDK 1.5及以后的版本中，会转化为StringBuidler对象的连续append()操作。
```java
    public static String test03(String s1, String s2, String s3) {
        String s = s1 + s2 + s3;
        return s;
    }
```
上述代码使用javap 编译结果

![](/images/current/synchronized/schronized-xiaochu.png "锁消除")

众所周知，StringBuilder不是安全同步的，但是在上述代码中，JVM判断该段代码并不会逃逸，则将该代码带默认为线程独有的资源，并不需要同步，所以执行了锁消除操作。(还有Vector中的各种操作也可实现锁消除。在没有逃逸出数据安全防卫内) 


### 锁粗化
通常情况下，为了保证多线程间的有效并发，会要求每个线程持有锁的时间尽可能短，但是在某些情况下，一个程序对同一个锁不间断、高频地请求、同步与释放，会消耗掉一定的系统资源，因为锁的请求、同步与释放本身会带来性能损耗，这样高频的锁请求就反而不利于系统性能的优化了，虽然单次同步操作的时间可能很短。锁粗化就是告诉我们任何事情都有个度，有些情况下我们反而希望把很多次锁的请求合并成一个请求，以降低短时间内大量锁请求、同步、释放带来的性能损耗。

![](/images/current/JCP-LockCH.png "锁粗化")


## 锁详细变迁过程（扩展掌握）
在Java SE 1.6里Synchronied同步锁，一共有四种状态：无锁、偏向锁、轻量级所、重量级锁，它会随着竞争情况逐渐升级。

锁可以升级但是不可以降级，目的是为了提供获取锁和释放锁的效率。 

**锁膨胀方向(不可逆)： 无锁 → 偏向锁 → 轻量级锁 → 重量级锁** 

**流程：偏向锁通过对比Mark Word解决加锁问题，避免执行CAS操作。而轻量级锁是通过用CAS操作和自旋来解决加锁问题，避免线程阻塞和唤醒而影响性能。重量级锁是将除了拥有锁的线程以外的线程都阻塞。**

下面是各个过程的详细介绍：

### 自旋锁与自适应自旋锁
![](/images/current/synchronized/sync-zixuan.png "自旋锁对比非自旋锁")

#### 自旋锁
> 引入背景：大家都知道，在没有加入锁优化时，Synchronized是一个非常“胖大”的家伙。在多线程竞争锁时，当一个线程获取锁时，它会阻塞所有正在竞争的线程，这样对性能带来了极大的影响。在挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作对系统的并发性能带来了很大的压力。同时HotSpot团队注意到在很多情况下，共享数据的**锁定状态只会持续很短的一段时间，为了这段时间去挂起和恢复阻塞线程并不值得**。在如今多处理器环境下，完全可以让另一个**没有获取到锁的线程在门外等待一会(自旋)，但不放弃CPU的执行时间**。等待持有锁的线程是否很快就会释放锁。**为了让线程等待，我们只需要让线程执行一个忙循环(自旋)，这便是自旋锁由来的原因。**

自旋锁早在JDK1.4 中就引入了，只是当时默认时关闭的。在JDK 1.6后默认为开启状态。

自旋锁本质上与阻塞并不相同，先不考虑其对多处理器的要求，如果锁占用的时间非常的短，那么自旋锁的性能会非常的好，相反，其会带来更多的性能开销(因为在线程自旋时，始终会占用CPU的时间片，如果锁占用的时间太长，那么自旋的线程会白白消耗掉CPU资源)。

因此自旋等待的时间必须要有一定的限度，如果自选超过了限定的次数仍然没有成功获取到锁，就应该使用传统的方式去挂起线程了，在JDK定义中，自旋锁默认的自旋次数为10次，用户可以使用参数-XX:PreBlockSpin来更改。 

可是现在又出现了一个问题：如果线程锁在线程自旋刚结束就释放掉了锁，那么是不是有点得不偿失。所以这时候我们需要更加聪明的锁来实现更加灵活的自旋。来提高并发的性能。(这里则需要自适应自旋锁) 

#### 自适应自旋锁
在JDK 1.6中引入了自适应自旋锁。这就意味着自旋的时间不再固定了，而是**由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定的**。

如果在同一个锁对象上，自旋等待刚刚成功获取过锁，并且持有锁的线程正在运行中，那么JVM会认为该锁自旋获取到锁的可能性很大，会自动增加等待时间。

比如增加到100此循环。相反，如果对于某个锁，自旋很少成功获取锁。那再以后要获取这个锁时将可能省略掉自旋过程，以避免浪费处理器资源。

有了**自适应自旋，JVM对程序的锁的状态预测会越来越准备，JVM也会越来越聪明**。 

### 偏向锁
> 引入背景：在大多实际环境下，锁不仅不存在多线程竞争，而且总是由同一个线程多次获取，那么在同一个线程反复获取所释放锁中，其中并还没有锁的竞争，那么这样看上去，多次的获取锁和释放锁带来了很多不必要的性能开销和上下文切换。 ​

为了解决这一问题，HotSpot的作者在Java SE1.6中对Synchronized进行了优化，引入了偏向锁。

当一个线程访问同步快并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁。只需要简单地测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁。如果成功，表示线程已经获取到了锁。

![](/images/current/synchronized/pianxiang1.png "偏向锁1")

#### 偏向锁的撤销
**偏向锁使用了一种等待竞争出现才会释放锁的机制，所以当其他线程尝试获取偏向锁时，持有偏向锁的线程才会释放锁。**

但是偏向锁的撤销需要等到全局安全点(就是当前线程没有正在执行的字节码)。

它会首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否活着。

如果线程不处于活动状态，直接将对象头设置为无锁状态。

如果线程活着，JVM会遍历栈帧中的锁记录，栈帧中的锁记录和对象头要么偏向于其他线程，要么恢复到无锁状态或者标记对象不适合作为偏向锁。

![](/images/current/synchronized/pianxiang2.png "偏向锁2")

### 轻量级锁
> 引入背景：这种锁实现的背后基于这样一种假设，即在真实的情况下我们程序中的大部分同步代码一般都处于无锁竞争状态(即单线程执行环境)，在无锁竞争的情况下完全可以**避免调用操作系统层面的重量级互斥锁**，取而代之的是在**monitorenter和monitorexit中只需要依靠一条CAS原子指令就可以完成锁的获取及释放**。当存在锁竞争的情况下，执行CAS指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒。

​在JDK 1.6之后引入的轻量级锁，需要注意的是轻量级锁并不是替代重量级锁的，而是对在大多数情况下同步块并不会有竞争出现提出的一种优化。
**它可以减少重量级锁对线程的阻塞带来地线程开销，从而提高并发性能。** 

​如果要理解轻量级锁，那么必须先要了解HotSpot虚拟机中对象头的内存布局。

在对象头中(Object Header)存在两部分：
* 第一部分用于存储对象自身的运行时数据，HashCode、GCAge、锁标记位、是否为偏向锁等。一般为32位或者64位(视操作系统位数定)。官方称之为Mark Word，它是实现轻量级锁和偏向锁的关键。
* 另外一部分存储的是指向方法区对象类型数据的指针(Klass Point)，如果对象是数组的话，还会有一个额外的部分用于存储数据的长度。 

#### 轻量级锁加锁 
在线程执行同步块之前，JVM会先在**当前线程的栈帧中创建一个名为锁记录(Lock Record)的空间**，用于存储锁对象目前的Mark Word的拷贝(JVM会将对象头中的Mark Word拷贝到锁记录中，官方称为Displaced Mark Ward)这个时候线程堆栈与对象头的状态如图：

![](/images/current/synchronized/Lightweight-1.png "轻量级锁加锁1")

如上图所示：如果当前对象没有被锁定，那么锁标志位位01状态，JVM在执行当前线程时，首先会在当前线程栈帧中创建锁记录Lock Record的空间用于存储锁对象目前的Mark Word的拷贝。 ​ 

然后，**虚拟机使用CAS操作将标记字段Mark Word拷贝到锁记录中，并且将Mark Word更新为指向Lock Record的指针**。如果更新成功了，那么这个线程就有用了该对象的锁，并且对象Mark Word的锁标志位更新为(Mark Word中最后的2bit)00，即表示此对象处于轻量级锁定状态，如图：

![](/images/current/synchronized/Lightweight-2.png "轻量级锁加锁2")

如果这个更新操作失败，JVM会检查当前的Mark Word中是否存在指向当前线程的栈帧的指针，如果有，说明该锁已经被获取，可以直接调用。如果没有，则说明该锁被其他线程抢占了。

**如果有两条以上的线程竞争同一个锁，那轻量级锁就不再有效，直接膨胀位重量级锁，没有获得锁的线程会被阻塞**。此时，锁的标志位为10，Mark Word中存储的是指向重量级锁的指针。 ​ 
 
轻量级解锁时，会使用原子的CAS操作将Displaced Mark Word替换回到对象头中，如果成功，则表示没有发生竞争关系。如果失败，表示当前锁存在竞争关系。锁就会膨胀成重量级锁。两个线程同时争夺锁，导致锁膨胀的流程图如下：

![](/images/current/synchronized/Lightweight-3.png "轻量级锁加锁3")