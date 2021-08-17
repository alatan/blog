# MySQL高可用方案


## MySQL主从复制架构
![](/images/db/ha/ms.png "MySQL主从架构")

### 基本原理
slave 会从 master 读取 binlog 来进行数据同步
### 三个步骤
* master将改变记录到二进制日志（binary log）。这些记录过程叫做二进制日志事件，binary log events；
* salve 将 master 的 binary log events 拷贝到它的中继日志（relay log）;
* slave 重做中继日志中的事件，将改变应用到自己的数据库中。MySQL 复制是异步且是串行化的。

### 复制的基本原则
* 每个 slave只有一个 master
* 每个 salve只能有一个唯一的服务器 ID
* 每个master可以有多个salve

![](/images/db/ha/dbCopy.png "主从复制")
上图主从复制分了五个步骤进行：
1. 步骤一：主库的更新事件(update、insert、delete)被写到binlog
2. 步骤二：从库发起连接，连接到主库。
3. 步骤三：此时主库创建一个binlog dump thread，把binlog的内容发送到从库。
4. 步骤四：从库启动之后，创建一个I/O线程，读取主库传过来的binlog内容并写入到relay log
5. 步骤五：还会创建一个SQL线程，从relay log里面读取内容，从Exec_Master_Log_Pos位置开始执行读取到的更新事件，将更新内容写入到slave的db

### 此架构特点
1. 成本低，布署快速、方便
2. 读写分离
3. 还能通过及时增加从库来减少读库压力
4. **主库单点故障**
5. **数据一致性问题（同步延迟造成）**

## MySQL+MHA架构
> MHA目前在Mysql高可用方案中应该也是比较成熟和常见的方案，它由日本人开发出来，在mysql故障切换过程中，MHA能做到快速自动切换操作，而且还能最大限度保持数据的一致性。

![](/images/db/ha/MHA.png "MySQL+MHA架构")

该软件由两部分组成：MHA Manager（管理节点）和MHA Node（数据节点）。MHA Manager可以单独部署在一台独立的机器上管理多个master-slave集群，也可以部署在一台slave节点上。MHA Node运行在每台MySQL服务器上，MHA Manager会定时探测集群中的master节点，当master出现故障时，它可以自动将最新数据的slave提升为新的master，然后将所有其他的slave重新指向新的master。整个故障转移过程对应用程序完全透明。

在MHA自动故障切换过程中，MHA试图从宕机的主服务器上保存二进制日志，最大程度的保证数据的不丢失(配合mysql半同步复制效果更佳)，但这并不总是可行的。例如，如果主服务器硬件故障或无法通过ssh访问，MHA没法保存二进制日志，只进行故障转移而丢失了最新的数据。使用MySQL 5.5的半同步复制，可以大大降低数据丢失的风险。MHA可以与半同步复制结合起来。如果只有一个slave已经收到了最新的二进制日志，MHA可以将最新的二进制日志应用于其他所有的slave服务器上，因此可以保证所有节点的数据一致性。

注意：目前MHA主要支持一主多从的架构，要搭建MHA,要求一个复制集群中必须最少有三台数据库服务器，一主二从，即一台充当master，一台充当备用master，另外一台充当从库，因为至少需要三台服务器，出于机器成本的考虑，淘宝也在该基础上进行了改造，目前淘宝TMHA已经支持一主一从。


## 常见问题
### 百万级别或以上的数据如何删除
关于索引：由于索引需要额外的维护成本，因为索引文件是单独存在的文件,所以当我们对数据的增加,修改,删除,都会产生额外的对索引文件的操作,这些操作需要消耗额外的IO,会降低增/改/删的执行效率。所以，在我们删除数据库百万级别数据的时候，查询MySQL官方手册得知删除数据的速度和创建的索引数量是成正比的。

1. 所以我们想要删除百万数据的时候可以先删除索引（此时大概耗时三分多钟）
1. 然后删除其中无用数据（此过程需要不到两分钟）
1. 删除完成后重新创建索引(此时数据较少了)创建索引也非常快，约十分钟左右。
1. 与之前的直接删除绝对是要快速很多，更别说万一删除中断,一切删除会回滚。那更是坑了。


## 参考文章
* [浅谈MySQL集群高可用架构](https://segmentfault.com/a/1190000020200096 "浅谈MySQL集群高可用架构")
* [MySQL 同步复制及高可用方案总结](https://segmentfault.com/a/1190000022313462 "MySQL同步复制及高可用方案总结")
* [MySQL应用架构演变](https://segmentfault.com/a/1190000039693053 " MySQL应用架构演变")
* [Mysql高可用架构之keepalived and MHA ](https://kim1024.github.io/2018/11/20/mysql-keepalived-mha.html " Mysql高可用架构之keepalived and MHA ")

