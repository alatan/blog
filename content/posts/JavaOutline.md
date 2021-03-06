---  
title: "Java知识大纲"  
date: 2020-01-01
weight: 70  
draft: false  
keywords: [""]  
description: "Java知识大纲"  
tags: ["大纲"]  
categories: ["知识体系"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---  
## 基础
### 数据结构与算法

### 操作系统

### 网络基础

### 数据库基础

### 编译原理

## Java
### Java基础

### 并发编程

### JVM

### 性能优化
* 性能指标体系
* JVM调优
* Tomcat调优
* MySQL调优

### 故障排除

## 最佳实践

### 重构

### 设计模式

### 开发框架
* Spring体系
* MyBatis

### 常见业务
* 支付幂等性
* 减库存
* 秒杀
* 分布式锁
    redis实现的分布式锁。
    1. 应该保证互斥性（在任何时候只有一个客户端持有锁。使用setnx）。
    2. 不能死锁（设置过期时间）。
    3. 保证上锁和解锁是同一个客户端（设置不同的value值）。
    4. 业务时间太长。导致锁过期（设置看门狗。自动续锁）。
    5. 锁的重入性（使用redis的hset）。
* 分布式事务
* 分布式缓存

## 中间价
### 消息队列

### 缓存
* 本地缓存
* 分布式缓存

### ELK

### 数据库
* 分库分表
* 数据同步
* 数据库连接池

## 分布式
分布式架构原理
分布式架构策略
分布式中间件
分布式架构实战
### 四大理论
* 拜占庭将军问题
* CAP 理论
* ACID 理论
* BASE 理论

### 八大协议/算法
* Paxos 算法
* Raft 算法
* 一致性 Hash 算法
* Gossip 协议算法
* Quorum NWR 算法
* FBFT 算法
* POW 算法
* ZAB 协议

## 微服务

## 工具
* 版本管理 Git
* 项目管理 Maven/Gradle
* 代码质量管理 Sonar
* 持续集成部署 Jenkins&GitLab CI/CD
* 监控系统
* 测试
    * Postman
    * Jmeter
    * VisualVM