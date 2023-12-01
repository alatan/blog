# Kafka

> Kafka 是一个分布式消息引擎与流处理平台，经常用做企业的消息总线、实时数据管道，有的还把它当做存储系统来使用。早期 Kafka 的定位是一个高吞吐的分布式消息系统，目前则演变成了一个成熟的分布式消息引擎，以及流处理平台。


> 使用消息队列不可能是单机的（必然是分布式or集群）

Kafka天然是分布式的，往一个topic丢数据，实际上就是往多个broker的partition存储数据

> 数据写到消息队列，可能会存在数据丢失问题，数据在消息队列需要持久化

Kafka会将partition以消息日志的方式(落磁盘)存储起来，通过 顺序访问IO和缓存(等到一定的量或时间)才真正把数据写到磁盘上，来提高速度。

> 想要保证消息（数据）是有序的，怎么做？

Kafka会将数据写到partition，单个partition的写入是有顺序的。如果要保证全局有序，那只能写入一个partition中。如果要消费也有序，消费者也只能有一个。

## Kafka术语
* Producer：生产者，消息产生和发送端。
* Broker：Kafka 实例，多个 broker 组成一个 Kafka 集群，通常一台机器部署一个 Kafka 实例，一个实例挂了不影响其他实例。
* Consumer：消费者，拉取消息进行消费。 一个 topic 可以让若干个消费者进行消费，若干个消费者组成一个 Consumer Group 即消费组，一条消息只能被消费组中一个 Consumer 消费。
* Topic：主题，服务端消息的逻辑存储单元。一个 topic 通常包含若干个 Partition 分区。
* Partition：topic 的分区，分布式存储在各个 broker 中， 实现发布与订阅的负载均衡。若干个分区可以被若干个 Consumer 同时消费，达到消费者高吞吐量。一个分区拥有多个副本（Replica），这是Kafka在可靠性和可用性方面的设计，后面会重点介绍。
* message：消息，或称日志消息，是 Kafka 服务端实际存储的数据，每一条消息都由一个 key、一个 value 以及消息时间戳 timestamp 组成。
* offset：偏移量，分区中的消息位置，由 Kafka 自身维护，Consumer 消费时也要保存一份 offset 以维护消费过的消息位置。

## Kafka特点
* 高吞吐、低延时：这是 Kafka 显著的特点，Kafka 能够达到百万级的消息吞吐量，延迟可达毫秒级；
* 持久化存储：Kafka 的消息最终持久化保存在磁盘之上，提供了顺序读写以保证性能，并且通过 Kafka 的副本机制提高了数据可靠性。
* 分布式可扩展：Kafka 的数据是分布式存储在不同 broker 节点的，以 topic 组织数据并且按 partition 进行分布式存储，整体的扩展性都非常好。
* 高容错性：集群中任意一个 broker 节点宕机，Kafka 仍能对外提供服务。

## Kafka消息发送机制
### 异步发送
Kafka 自从 0.8.2 版本就引入了新版本 Producer API，新版 Producer 完全是采用异步方式发送消息。生产端构建的 ProducerRecord 先是经过 keySerializer、valueSerializer 序列化后，再是经过 Partition 分区器处理，决定消息落到 topic 具体某个分区中，最后把消息发送到客户端的消息缓冲池 accumulator 中，交由一个叫作 Sender 的线程发送到 broker 端。

这里缓冲池 accumulator 的最大大小由参数 buffer.memory 控制，默认是 32M，当生产消息的速度过快导致 buffer 满了的时候，将阻塞 max.block.ms 时间，超时抛异常，所以 buffer 的大小可以根据实际的业务情况进行适当调整。

### 批量发送
发送到缓冲 buffer 中消息将会被分为一个一个的 batch，分批次的发送到 broker 端，批次大小由参数 batch.size 控制，默认16KB。这就意味着正常情况下消息会攒够 16KB 时才会批量发送到 broker 端，所以一般减小 batch 大小有利于降低消息延时，增加 batch 大小有利于提升吞吐量。

那么生成端消息是不是必须要达到一个 batch 大小时，才会批量发送到服务端呢？答案是否定的，Kafka 生产端提供了另一个重要参数 linger.ms，该参数控制了 batch 最大的空闲时间，超过该时间的 batch 也会被发送到 broker 端。

### 消息重试
此外，Kafka 生产端支持重试机制，对于某些原因导致消息发送失败的，比如网络抖动，开启重试后 Producer 会尝试再次发送消息。该功能由参数 retries 控制，参数含义代表重试次数，默认值为 0 表示不重试，建议设置大于 0 比如 3。

## Kafka 副本机制
> 副本机制也称 Replication 机制是 Kafka 实现高可靠、高可用的基础。Kafka 中有 leader 和 follower 两类副本。

### Kafka 副本作用
Kafka 默认只会给分区设置一个副本，由 broker 端参数 default.replication.factor 控制，默认值为 1，通常我们会修改该默认值，或者命令行创建 topic 时指定 replication-factor 参数，生产建议设置 3 副本。副本作用主要有两方面：
* 消息冗余存储，提高 Kafka 数据的可靠性；
* 提高 Kafka 服务的可用性，follower 副本能够在 leader 副本挂掉或者 broker 宕机的时候参与 leader 选举，继续对外提供读写服务。

### 关于读写分离
这里要说明的是 Kafka 并不支持读写分区，生产消费端所有的读写请求都是由 leader 副本处理的，follower 副本的主要工作就是从 leader 副本处异步拉取消息，进行消息数据的同步，并不对外提供读写服务。

Kafka 之所以这样设计，主要是为了保证读写一致性，因为副本同步是一个异步的过程，如果当 follower 副本还没完全和 leader 同步时，从 follower 副本读取数据可能会读不到最新的消息。

### ISR 副本集合
Kafka 为了维护分区副本的同步，引入 ISR（In-Sync Replicas ）副本集合的概念，ISR 是分区中正在与 leader 副本进行同步的 replica 列表，且必定包含 leader 副本。

ISR 列表是持久化在 Zookeeper 中的，任何在 ISR 列表中的副本都有资格参与 leader 选举。

ISR 列表是动态变化的，并不是所有的分区副本都在 ISR 列表中，哪些副本会被包含在 ISR 列表中呢？副本被包含在 ISR 列表中的条件是由参数 replica.lag.time.max.ms 控制的，参数含义是副本同步落后于 leader 的最大时间间隔，默认10s，意思就是说如果某一 follower 副本中的消息比 leader 延时超过10s，就会被从 ISR 中排除。Kafka 之所以这样设计，主要是为了减少消息丢失，只有与 leader 副本进行实时同步的 follower 副本才有资格参与 leader 选举，这里指相对实时。

### Unclean leader 选举
既然 ISR 是动态变化的，所以 ISR 列表就有为空的时候，ISR 为空说明 leader 副本也“挂掉”了，此时 Kafka 就要重新选举出新的 leader。但 ISR 为空，怎么进行 leader 选举呢？
Kafka 把不在 ISR 列表中的存活副本称为“非同步副本”，这些副本中的消息远远落后于 leader，如果选举这种副本作为 leader 的话就可能造成数据丢失。Kafka broker 端提供了一个参数 unclean.leader.election.enable，用于控制是否允许非同步副本参与 leader 选举；如果开启，则当 ISR 为空时就会从这些副本中选举新的 leader，这个过程称为 Unclean leader 选举。

前面也提及了，如果开启 Unclean leader 选举，可能会造成数据丢失，但保证了始终有一个 leader 副本对外提供服务；如果禁用 Unclean leader 选举，就会避免数据丢失，但这时分区就会不可用。这就是典型的 CAP 理论，即一个系统不可能同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition Tolerance）中的两个。所以在这个问题上，Kafka 赋予了我们选择 C 或 A 的权利。

我们可以根据实际的业务场景选择是否开启 Unclean leader选举，这里建议关闭 Unclean leader 选举，因为通常数据的一致性要比可用性重要的多。

## Kafka控制器
> 控制器（Controller）是 Kafka 的核心组件，它的主要作用是在 Zookeeper 的帮助下管理和协调整个 Kafka 集群。集群中任意一个 broker 都能充当控制器的角色，但在运行过程中，只能有一个 broker 成为控制器。

这里先介绍下 Zookeeper，因为控制器的产生依赖于 Zookeeper 的 ZNode 模型和 Watcher 机制。Zookeeper 的数据模型是类似 Unix 操作系统的 ZNode Tree 即 ZNode 树，ZNode 是 Zookeeper 中的数据节点，是 Zookeeper 存储数据的最小单元，每个 ZNode 可以保存数据，也可以挂载子节点，根节点是 /。基本的拓扑图如下：

Zookeeper 有两类 ZNode 节点，分别是持久性节点和临时节点。持久性节点是指客户端与 Zookeeper 断开会话后，该节点依旧存在，直到执行删除操作才会清除节点。临时节点的生命周期是和客户端的会话绑定在一起，客户端与 Zookeeper 断开会话后，临时节点就会被自动删除。

Watcher 机制是 Zookeeper 非常重要的特性，它可以在 ZNode 节点上绑定监听事件，比如可以监听节点数据变更、节点删除、子节点状态变更等事件，通过这个事件机制，可以基于 ZooKeeper 实现分布式锁、集群管理等功能。

### 控制器选举
当集群中的任意 broker 启动时，都会尝试去 Zookeeper 中创建 /controller 节点，第一个成功创建 /controller 节点的 broker 则会被指定为控制器，其他 broker 则会监听该节点的变化。当运行中的控制器突然宕机或意外终止时，其他 broker 能够快速地感知到，然后再次尝试创建 /controller 节点，创建成功的 broker 会成为新的控制器。

### 控制器功能
前面我们也说了，控制器主要作用是管理和协调 Kafka 集群，那么 Kafka 控制器都做了哪些事情呢，具体如下：
* 主题管理：创建、删除 topic，以及增加 topic 分区等操作都是由控制器执行。
* 分区重分配：执行 Kafka 的 reassign 脚本对 topic 分区重分配的操作，也是由控制器实现。
* Preferred leader 选举：这里有一个概念叫 Preferred replica 即优先副本，表示的是分配副本中的第一个副本。Preferred leader 选举就是指 Kafka 在某些情况下出现 leader 负载不均衡时，会选择 preferred 副本作为新 leader 的一种方案。这也是控制器的职责范围。
* 集群成员管理：控制器能够监控新 broker 的增加，broker 的主动关闭与被动宕机，进而做其他工作。这里也是利用前面所说的 Zookeeper 的 ZNode 模型和 Watcher 机制，控制器会监听 Zookeeper 中 /brokers/ids 下临时节点的变化。
* 数据服务：控制器上保存了最全的集群元数据信息，其他所有 broker 会定期接收控制器发来的元数据更新请求，从而更新其内存中的缓存数据。

从上面内容我们大概知道，控制器可以说是 Kafka 的心脏，管理和协调着整个 Kafka 集群，因此控制器自身的性能和稳定性就变得至关重要。

社区在这方面做了大量工作，特别是在 0.11 版本中对控制器进行了重构，其中最大的改进把控制器内部多线程的设计改成了单线程加事件队列的方案，消除了多线程的资源消耗和线程安全问题，另外一个改进是把之前同步操作 Zookeeper 改为了异步操作，消除了 Zookeeper 端的性能瓶颈，大大提升了控制器的稳定性。

## Kafka消费端Rebalance机制
> 就 Kafka 消费端而言，有一个难以避免的问题就是消费者的重平衡即 Rebalance。Rebalance 是让一个消费组的所有消费者就如何消费订阅 topic 的所有分区达成共识的过程，在 Rebalance 过程中，所有 Consumer 实例都会停止消费，等待 Rebalance 的完成。因为要停止消费等待重平衡完成，因此 Rebalance 会严重影响消费端的 TPS，是应当尽量避免的。

### Rebalance 发生条件
关于何时会发生 Rebalance，总结起来有三种情况：
* 消费组的消费者成员数量发生变化
* 消费主题的数量发生变化
* 消费主题的分区数量发生变化

其中后两种情况一般是计划内的，比如为了提高消息吞吐量增加 topic 分区数，这些情况一般是不可避免的，后面我们会重点讨论如何避免因为组内消费者成员数发生变化导致的 Rebalance。

### Kafka协调器
在介绍如何避免 Rebalance 问题之前，先来认识下 Kafka 的协调器 Coordinator，和之前 Kafka 控制器类似，Coordinator 也是 Kafka 的核心组件。
主要有两类 Kafka 协调器：
* 组协调器（Group Coordinator）
* 消费者协调器（Consumer Coordinator）

Kafka 为了更好的实现消费组成员管理、位移管理，以及 Rebalance 等，broker 服务端引入了组协调器（Group Coordinator），消费端引入了消费者协调器（Consumer Coordinator）。每个 broker 启动的时候，都会创建一个 GroupCoordinator 实例，负责消费组注册、消费者成员记录、offset 等元数据操作，这里也可以看出每个 broker 都有自己的 Coordinator 组件。另外，每个 Consumer 实例化时，同时会创建一个 ConsumerCoordinator 实例，负责消费组下各个消费者和服务端组协调器之前的通信。可以用下图表示协调器原理：

客户端的消费者协调器 Consumer Coordinator 和服务端的组协调器 Group Coordinator 会通过心跳不断保持通信。

### 如何避免消费组 Rebalance
接下来我们讨论下如何避免组内消费者成员发生变化导致的 Rebalance。组内成员发生变化无非就两种情况，一种是有新的消费者加入，通常是我们为了提高消费速度增加了消费者数量，比如增加了消费线程或者多部署了一份消费程序，这种情况可以认为是正常的；另一种是有消费者退出，这种情况多是和我们消费端代码有关，是我们要重点避免的。

正常情况下，每个消费者都会定期向组协调器 Group Coordinator 发送心跳，表明自己还在存活，如果消费者不能及时的发送心跳，组协调器会认为该消费者已经“死”了，就会导致消费者离组引发 Rebalance 问题。这里涉及两个消费端参数：session.timeout.ms 和 heartbeat.interval.ms，含义分别是组协调器认为消费组存活的期限，和消费者发送心跳的时间间隔，其中 heartbeat.interval.ms 默认值是3s，session.timeout.ms 在 0.10.1 版本之前默认 30s，之后默认 10s。

另外，0.10.1 版本还有两个值得注意的地方：
* 从该版本开始，Kafka 维护了单独的心跳线程，之前版本中 Kafka 是使用业务主线程发送的心跳。
* 增加了一个重要的参数 max.poll.interval.ms，表示 Consumer 两次调用 poll 方法拉取数据的最大时间间隔，默认值 5min，对于那些忙于业务逻辑处理导致超过 max.poll.interval.ms 时间的消费者将会离开消费组，此时将发生一次 Rebalance。

此外，如果 Consumer 端频繁 FullGC 也可能会导致消费端长时间停顿，从而引发 Rebalance。因此，我们总结如何避免消费组 Rebalance 问题，主要从以下几方面入手：
* 合理配置 session.timeout.ms 和 heartbeat.interval.ms，建议 0.10.1 之前适当调大 session 超时时间尽量规避 Rebalance。
* 根据实际业务调整 max.poll.interval.ms，通常建议调大避免 Rebalance，但注意 0.10.1 版本之前没有该参数。
* 监控消费端的 GC 情况，避免由于频繁 FullGC 导致线程长时间停顿引发 Rebalance。

合理调整以上参数，可以减少生产环境中 Rebalance 发生的几率，提升 Consumer 端的 TPS 和稳定性。

## 常见问题
### Kafka 为什么这么快
Kafka 的消息是保存或缓存在磁盘上的，一般认为在磁盘上读写数据是会降低性能的，因为寻址会比较消耗时间，但是实际上，Kafka 的特性之一就是高吞吐率。Kafka 之所以能这么快，无非是：「**顺序写磁盘、大量使用内存页 、零拷贝技术的使用**」

它把所有的消息都变成一个批量的文件，并且进行合理的批量压缩，减少网络 IO 损耗，通过 mmap 提高 I/O 速度。写入数据的时候由于单个 Partion 是末尾添加，所以速度最优；读取数据的时候配合 Sendfile 直接暴力输出。

## 参考链接
* [深入浅出Kafka](https://mp.weixin.qq.com/s/-IPfWPS1WQMEgcIu0Ak2VQ)
* [Kafka分区和副本](https://www.cnblogs.com/listenfwind/p/12465409.html)
* [Kafka原理和实践](https://juejin.cn/post/6844903919424913415)
* [Kafka整理](https://mp.weixin.qq.com/s/9fJchPJa_raHSkvo29bkEA)
* [Kafka为什么快](https://mp.weixin.qq.com/s/ejZBAGI7qLE_QYSe-AqipA)

