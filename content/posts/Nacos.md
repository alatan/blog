--- 
title: "Nacos解析"  
date: 2020-04-11
weight: 70  
draft: false  
keywords: ["SpringCloud"]  
description: "Nacos解析"  
author: "默哥"  

categories: ["SpringCloud"]
tags: ["SpringCloud", "微服务"]  
toc: 
    auto: false
---

## 临时实例和持久化实例。
* 在定义上区分临时实例和持久化实例的关键是健康检查的方式。
* 临时实例使用客户端上报模式，而持久化实例使用服务端反向探测模式。
* 临时实例需要能够自动摘除不健康实例，而且无需持久化存储实例，那么这种实例就适用于类 Gossip 的协议。
* 右边的持久化实例使用服务端探测的健康检查方式，因为客户端不会上报心跳，那么自然就不能去自动摘除下线的实例。

## 实现对比 
* Zookeeper：保证CP，放弃可用性；一旦zookeeper集群中master节点宕了，则会重新选举leader，这个过程可能非常漫长，在这过程中服务不可用。
* Eureka：保证AP，放弃一致性；Eureka集群中的各个节点都是平等的，一旦某个节点宕了，其他节点正常服务（一旦客户端发现注册失败，则将会连接集群中其他节点），虽然保证了可用性，但是每个节点的数据可能不是最新的。
* Nacos：同时支持CP和AP，默认是AP，可以切换；AP模式下以临时实例注册，CP模式下服务永久实例注册。

## 健康检查
### 客户端健康检查
客户端健康检查主要关注客户端上报心跳的方式、服务端摘除不健康客户端的机制。
### 服务端健康检查
而服务端健康检查，则关注探测客户端的方式、灵敏度及设置客户端健康状态的机制。

从实现复杂性来说，服务端探测肯定是要更加复杂的，因为需要服务端根据注册服务配置的健康检查方式，去执行相应的接口，判断相应的返回结果，并做好重试机制和线程池的管理。

这与客户端探测，只需要等待心跳，然后刷新 TTL 是不一样的。同时服务端健康检查无法摘除不健康实例，这意味着只要注册过的服务实例，如果不调用接口主动注销，这些服务实例都需要去维持健康检查的探测任务，而客户端则可以随时摘除不健康实例，减轻服务端的压力。

## Nacos实现配置管理和动态配置
1. 添加对应spring-cloud-starter-alibaba-nacos-config依赖
1. 使用原生注解@Value()导入配置
1. 使用原生注解@RefreshScope刷新配置
1. 根据自己业务场景做好多环境配置隔离(Namespace)、不同业务配置隔离(Group)
1. 切记：命名空间和分组的配置一定要放在bootstrap.yml或者bootstrap.properties配置文件中

## 参考文章
* [Nacos 注册中心的设计原理详解](https://www.infoq.cn/article/b*6vymikao9vakisjype)
* [Nacos 实现原理详解](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247525804&idx=3&sn=3499e1b228b85efb2770b8bcac7b47d7)
* [Nacos 使用](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247525731&idx=3&sn=800a591510807c9d43ab542658cfc3d9)