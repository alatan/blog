# Java知识大纲

  
## 基础
### 数据结构与算法
* 数组、链表、二叉树、队列、栈的各种操作（性能，场景）
* 二分查找和各种变种的二分查找
* 各类排序算法以及复杂度分析（快排、归并、堆）
* 各类算法题（手写）
* 理解并可以分析时间和空间复杂度。
* 动态规划（笔试回回有。。）、贪心。
* 红黑树、AVL树、Hash树、Tire树、B树、B+树。
* 图算法（比较少，也就两个最短路径算法理解吧）
### 操作系统
* 进程通信IPC（几种方式），与线程区别
* OS的几种策略（页面置换，进程调度等，每个里面有几种算法）
* 互斥与死锁相关的
* linux常用命令（问的时候都会给具体某一个场景）
* Linux内核相关（select、poll、epoll）
### 网络基础
#### OSI7层模型（TCP4层）
* 每层的协议
* url到页面的过程
#### HTTP
* http/https 1.0、1.1、2.0
* get/post 以及幂等性
* http 协议头相关
网络攻击（CSRF、XSS）
#### TCP/IP
* 三次握手、四次挥手
* 拥塞控制（过程、阈值）
* 流量控制与滑动窗口
* TCP与UDP比较
* 子网划分（一般只有笔试有）
* DDos攻击
#### (B)IO/NIO/AIO
* 三者原理，各个语言是怎么实现的
* Netty
* Linux内核select poll epoll
### 数据库
* 索引（包括分类及优化方式，失效条件，底层结构）
* sql语法（join，union，子查询，having，group by）
* 引擎对比（InnoDB，MyISAM）
* 数据库的锁（行锁，表锁，页级锁，意向锁，读锁，写锁，悲观锁，乐观锁，以及加锁的select sql方式）
* 隔离级别，依次解决的问题（脏读、不可重复读、幻读）
* 事务的ACID
* B树、B+树
* 优化（explain，慢查询，show profile）
* 数据库的范式。
* 分库分表，主从复制，读写分离。
* Nosql相关（redis和mem***d区别之类的，如果你熟悉redis，redis还有一堆要问的）
### 编译原理

## Java
### Java基础
* 把我之后的面经过一遍，Java感觉覆盖的就差不多了，不过下面还是分个类。
* Java基础（面向对象、四个特性、重载重写、static和final等等很多东西）
* 集合（HashMap、ConcurrentHashMap、各种List，最好结合源码看）
* 并发和多线程（线程池、SYNC和Lock锁机制、线程通信、volatile、ThreadLocal、CyclicBarrier、Atom包、CountDownLatch、AQS、CAS原理等等）
* JVM（内存模型、GC垃圾回收，包括分代，GC算法，收集器、类加载和双亲委派、JVM调优，内存泄漏和内存溢出）
* IO/NIO相关
* 反射和***、异常、Java8相关、序列化
* 设计模式（常用的，jdk中有的）
* Web相关（servlet、cookie/session、Spring<AOP、IOC、MVC、事务、动态***>、Mybatis、Tomcat、Hibernate等）
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
* CAP原理和BASE理论。
* Nosql与KV存储（redis，hbase，mongodb，mem***d等）
* 服务化理论（包括服务发现、治理等，zookeeper、etcd、springcloud微服务、）
* 负载均衡（原理、cdn、一致性hash）
* RPC框架（包括整体的一些框架理论，通信的netty，序列化协议thrift，protobuff等）
* 消息队列（原理、kafka，activeMQ，rocketMQ）
* 分布式存储系统（GFS、HDFS、fastDFS）、存储模型（skipList、LSM等）
* 分布式事务、分布式锁等
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

## 大数据与数据分析：
* hadoop生态圈(hive、hbase、hdfs、zookeeper、storm、kafka)
* spark体系
* 语言：python、R、scala
* 搜索引擎与技术


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
