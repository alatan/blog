# 分布式锁

## 基于数据库
* 基于数据库表数据记录做唯一约束

上面这种简单的实现有以下几个问题：
1. 这把锁强依赖数据库的可用性，数据库是一个单点，一旦数据库挂掉，会导致业务系统不可用。
2. 这把锁没有失效时间，一旦解锁操作失败，就会导致锁记录一直在数据库中，其他线程无法再获得到锁。
3. 这把锁只能是非阻塞的，因为数据的insert操作，一旦插入失败就会直接报错。没有获得锁的线程并不会进入排队队列，要想再次获得锁就要再次触发获得锁操作。
4. 这把锁是非重入的，同一个线程在没有释放锁之前无法再次获得该锁。因为数据中数据已经存在了。
5. 不过这种方式对于单主却无法自动切换主从的mysql来说，基本就无法现实P分区容错性，（Mysql自动主从切换在目前并没有十分完美的解决方案）。可以说这种方式强依赖于数据库的可用性，数据库写操作是一个单点，一旦数据库挂掉，就导致锁的不可用。这种方式基本不在CAP的一个讨论范围。

## 基于Redis
使用redis的setNX()用于分布式锁。（原子性）
``` javascript
setnx key value Expire_time
获取到锁 返回 1 ， 获取失败 返回 0
* 返回1，说明该进程获得锁，SETNX将键 lock.id 的值设置为锁的超时时间，当前时间 +加上锁的有效时间。
* 返回0，说明其他进程已经获得了锁，进程不能进入临界区。进程可以在一个循环中不断地尝试 SETNX 操作，以获得锁。
```

```java (jedis)
protected boolean getLock(String lockKey, int time,TimeUnit timeUnit) {
  // 使用redis的setNX命令实现分布式锁
  boolean isSuccess = redisTemplate.opsForValue().setIfAbsent(lockKey, "lock");
  // 防止进程中断导致redis分布锁死锁，检查是否设置了超时时间。
  Long timeLeft = redisTemplate.getExpire(lockKey);

  if (isSuccess || timeLeft < 0) {
    // 设置失效时间（最好与该任务执行频率时间一致）：如果执行期间宕机，5分钟后也能被另一机器获得lock
    redisTemplate.expire(lockKey, time, timeUnit);
  }
  return isSuccess;
}
```

为了解决数据库锁的主从切换的问题，可以选择redis集群，或者是 sentinel 哨兵模式，实现主从故障转移，当master节点出现故障，哨兵会从slave中选取节点，重新变成新的master节点。

哨兵模式故障转移是由sentinel集群进行监控判断，当maser出现异常即复制中止，重新推选新slave成为master，sentinel在重新进行选举并不保证主从数据复制完毕具备一致性。
所以redis的复制模式是属于AP的模式。保证可用性，在主从复制中“主”有数据，但是可能“从”还没有数据，这个时候，一旦主挂掉或者网络抖动等各种原因，可能会切换到“从”节点，这个时候可能会导致两个业务县城同时获取得两把锁

能不能使用redis作为分布式锁，这个本身就不是redis的问题，还是取决于业务场景，我们先要自己确认我们的场景是适合 AP 还是 CP ， 如果在社交发帖等场景下，我们并没有非常强的事务一致性问题，redis提供给我们高性能的AP模型是非常适合的，但如果是交易类型，对数据一致性非常敏感的场景，我们可能要寻在一种更加适合的 CP 模型

### Redis分布式锁如何续期，建议使用redisson客户端（可以自动续期）

## 基于ZooKeeper
> 每个客户端对某个方法加锁时，在zookeeper上的与该方法对应的指定节点的目录下，生成一个唯一的临时有序节点。 判断是否获取锁的方式很简单，只需要判断有序节点中序号最小的一个。 当释放锁的时候，只需将这个临时节点删除即可。同时，排队的节点需要监听排在自己之前的节点，这样能在节点释放时候接收到回调通知，让其获得锁。zk的session由客户端管理，其可以避免服务宕机导致的锁无
法释放，而产生的死锁问题，不需要关注锁超时。


刚刚也分析过，redis其实无法确保数据的一致性，先来看ZooKeeper是否合适作为我们需要的分布式锁，首先zk的模式是CP模型，也就是说，当zk锁提供给我们进行访问的时候，在zk集群中能确保这把锁在zk的每一个节点都存在。

### zk分布式锁的代码实现
zk官方提供的客户端并不支持分布式锁的直接实现，我们需要自己写代码去利用zk的这几个特性去进行实现。
![](/images/distributed/DL-zookeeper.jpg "zk分布式锁")

## 究竟该用CP还是AP的分布式锁
首先得了解清楚我们使用分布式锁的场景，为何使用分布式锁，用它来帮我们解决什么问题，先聊场景后聊分布式锁的技术选型。

无论是Redis，zk，例如Redis的AP模型会限制很多使用场景，但它却拥有了几者中最高的性能，ZooKeeper的分布式锁要比Redis可靠很多，但他繁琐的实现机制导致了它的性能不如Redis，而且zk会随着集群的扩大而性能更加下降。

## 参考文章
* [分布式锁，是选择AP还是选择CP](https://juejin.cn/post/6844903936718012430#heading-12)
* [分布式锁总结](https://juejin.cn/post/6844903726268809224)
* [基于 Redis 的分布式锁](https://juejin.cn/post/6844904126288150542)
* [Redis分布式锁如何续期](https://juejin.cn/post/6844903874675867656)
* [分布式锁高并发优化实践](https://juejin.cn/post/6844903719318847495)
