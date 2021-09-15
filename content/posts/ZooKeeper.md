---  
title: "ZooKeeper"  
date: 2020-03-15
weight: 70  
draft: false  
keywords: ["分布式"]  
description: "ZooKeeper"  
tags: [ "分布式"]  
categories: ["分布式"]  
author: "默哥"  

toc:
  auto: false
--- 
## 简介
* ZooKeeper主要**服务于分布式系统**，可以用ZooKeeper来做：统一配置管理、统一命名服务、分布式锁、集群管理。
* 使用分布式系统就无法避免对节点管理的问题(需要实时感知节点的状态、对节点进行统一管理等等)，而由于这些问题处理起来可能相对麻烦和提高了系统的复杂性，ZooKeeper作为一个能够**通用**解决这些问题的中间件就应运而生了。

## ZooKeeper 的一些重要概念
* ZooKeeper  本身就是一个分布式程序（只要半数以上节点存活，ZooKeeper  就能正常服务）。
* 为了保证高可用，最好是以集群形态来部署 ZooKeeper，这样只要集群中大部分机器是可用的（能够容忍一定的机器故障），那么 ZooKeeper 本身仍然是可用的。
* ZooKeeper  将数据保存在内存中，这也就保证了 高吞吐量和低延迟（但是内存限制了能够存储的容量不太大，此限制也是保持znode中存储的数据量较小的进一步原因）。
* ZooKeeper 是高性能的。 在“读”多于“写”的应用程序中尤其地高性能，因为“写”会导致所有的服务器间同步状态。（“读”多于“写”是协调服务的典型场景。）
* ZooKeeper有临时节点的概念。 当创建临时节点的客户端会话一直保持活动，瞬时节点就一直存在。而当会话终结时，瞬时节点被删除。持久节点是指一旦这个ZNode被创建了，除非主动进行ZN
移除操作，否则这个ZNode将一直保存在Zookeeper上。
* ZooKeeper 底层其实只提供了两个功能：
  * 管理（存储、读取）用户程序提交的数据；
  * 为用户程序提交数据节点监听服务。

## 可构建集群
为了保证高可用，最好是以集群形态来部署 ZooKeeper，这样只要集群中大部分机器是可用的（能够容忍一定的机器故障），那么zookeeper本身仍然是可用的。 客户端在使用 ZooKeeper 时，需要知道集群机器列表，通过与集群中的某一台机器建立 TCP 连接来使用服务，客户端使用这个TCP链接来发送请求、获取结果、获取监听事件以及发送心跳包。如果这个连接异常断开了，客户端可以连接到另外的机器上。

ZooKeeper 官方提供的架构图：
![](/images/distributed/zookeeper.png "ZooKeeper")
上图中每一个Server代表一个安装Zookeeper服务的服务器。组成 ZooKeeper 服务的服务器都会在内存中维护当前的服务器状态，并且每台服务器之间都互相保持着通信。集群间通过 Zab 协议（Zookeeper Atomic Broadcast）来保持数据的一致性。

## ZooKeeper 集群角色介绍

最典型集群模式： Master/Slave 模式（主备模式）。在这种模式中，通常 Master服务器作为主服务器提供写服务，其他的 Slave 服务器从服务器通过异步复制的方式获取 Master 服务器最新的数据提供读服务。

但是，在 ZooKeeper 中没有选择传统的  Master/Slave 概念，而是引入了Leader、Follower 和 Observer 三种角色。如下图所示
![](/images/distributed/ZookeeperCluster.png "ZookeeperCluster")

ZooKeeper 集群中的所有机器通过一个 Leader 选举过程来选定一台称为 “Leader” 的机器，Leader 既可以为客户端提供写服务又能提供读服务。除了 Leader 外，Follower 和  Observer 都只能提供读服务。Follower 和  Observer 唯一的区别在于 Observer 机器不参与 Leader 的选举过程，也不参与写操作的“过半写成功”策略，因此 Observer 机器可以在不影响写性能的情况下提升集群的读性能。

![](/images/distributed/zookeeperRole.png "zookeeperRole")

### 为什么最好使用奇数台服务器构成 ZooKeeper 集群
我们知道在Zookeeper中 Leader 选举算法采用了Zab协议。Zab核心思想是当多数 Server 写成功，则任务数据写成功。
1. 如果有3个Server，则最多允许1个Server 挂掉。
2. 如果有4个Server，则同样最多允许1个Server挂掉。
3. 既然3个或者4个Server，同样最多允许1个Server挂掉，那么它们的可靠性是一样的，所以选择奇数个ZooKeeper Server即可，这里选择3个Server。

## ZooKeeper &ZAB 协议&Paxos算法
Paxos 算法应该可以说是  ZooKeeper 的灵魂了。但是，ZooKeeper 并没有完全采用 Paxos算法 ，而是使用 ZAB 协议作为其保证数据一致性的核心算法。另外，在ZooKeeper的官方文档中也指出，ZAB协议并不像 Paxos 算法那样，是一种通用的分布式一致性算法，它是一种特别为Zookeeper设计的崩溃可恢复的原子消息广播算法。
###  ZAB 协议介绍
ZAB（ZooKeeper Atomic Broadcast 原子广播） 协议是为分布式协调服务 ZooKeeper 专门设计的一种支持崩溃恢复的原子广播协议。 在 ZooKeeper 中，主要依赖 ZAB 协议来实现分布式数据一致性，基于该协议，ZooKeeper 实现了一种主备模式的系统架构来保持集群中各个副本之间的数据一致性。

### ZAB 协议两种基本的模式：崩溃恢复和消息广播
ZAB协议包括两种基本的模式，分别是**崩溃恢复和消息广播**。当整个服务框架在启动过程中，或是当 Leader 服务器出现网络中断、崩溃退出与重启等异常情况时，ZAB 协议就会进人恢复模式并选举产生新的Leader服务器。当选举产生了新的 Leader 服务器，同时集群中已经有过半的机器与该Leader服务器完成了状态同步之后，ZAB协议就会退出恢复模式。其中，**所谓的状态同步是指数据同步，用来保证集群中存在过半的机器能够和Leader服务器的数据状态保持一致。**

**当集群中已经有过半的Follower服务器完成了和Leader服务器的状态同步，那么整个服务框架就可以进人消息广播模式了。**当一台同样遵守ZAB协议的服务器启动后加人到集群中时，如果此时集群中已经存在一个Leader服务器在负责进行消息广播，那么新加人的服务器就会自觉地进人数据恢复模式：找到Leader所在的服务器，并与其进行数据同步，然后一起参与到消息广播流程中去。正如上文介绍中所说的，ZooKeeper设计成只允许唯一的一个Leader服务器来进行事务请求的处理。Leader服务器在接收到客户端的事务请求后，会生成对应的事务提案并发起一轮广播协议；而如果集群中的其他机器接收到客户端的事务请求，那么这些非Leader服务器会首先将这个事务请求转发给Leader服务器。

关于 **ZAB 协议&Paxos算法** 需要讲和理解的东西太多了，说实话，笔主到现在不太清楚这俩兄弟的具体原理和实现过程。推荐阅读下面两篇文章：
* [图解 Paxos 一致性协议](http://blog.xiaohansong.com/2016/09/30/Paxos/)
* [Zookeeper ZAB 协议分析](http://blog.xiaohansong.com/2016/08/25/zab/)

关于如何使用 zookeeper 实现分布式锁，可以查看下面这篇文章：
* [Zookeeper ZAB 协议分析](https://blog.csdn.net/qiangcuo6087/article/details/79067136)

## 参考文章
* [zookepper概览](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247484868&idx=1&sn=af1e49c5f7dc89355255a4d46bafc005)
* [zookepper介绍](https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453140996&idx=1&sn=b2391f3eb780529020ace3a4c4357bda)

