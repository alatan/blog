# MySQL总结


![](/images/db/rdb/mysql-overview.png "MySQL结构")

## 简述一条SQL的执行流程
1. 客户端请求
2. 连接器（验证用户身份，给予权限） 
3. 查询缓存（存在缓存则直接返回，不存在则执行后续操作）
4. 分析器（对SQL进行词法分析和语法分析操作） 
5. 优化器（主要对执行的sql优化选择最优的执行方案方法） 
6. 执行器（执行时会先看用户是否有执行权限，有才去使用这个引擎提供的接口）
7. 去引擎层获取数据返回（如果开启查询缓存则会缓存查询结果）

![](/images/db/rdb/SQLLine.jpg "一条SQL的执行流程")

## 存储引擎
> 存储引擎是MySQL的组件，用于处理不同表类型的SQL操作。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能，使用不同的存储引擎，还可以获得特定的功能，MySQL服务器使用可插拔的存储引擎体系结构，可以从运行中的 MySQL 服务器加载或卸载存储引擎 。
### InnoDB 
> InnoDB 现在是 MySQL 默认的存储引擎，支持事务、行级锁定和外键，只有在需要它不支持的特性时，才考虑使用其它存储引擎。 

实现了四个标准的隔离级别，默认级别是可重复读(REPEATABLE READ)。在可重复读隔离级别下，通过多版本并发控制(MVCC)+ 间隙锁(Next-Key Locking)防止幻影读。 

主索引是聚簇索引，在索引中保存了数据，从而避免直接读取磁盘，因此对查询性能有很大的提升。 

内部做了很多优化，包括从磁盘读取数据时采用的可预测性读、能够加快读操作并且自动创建的自适应哈希索引、能够加速插入操作的插入缓冲区等。 

支持真正的在线热备份。其它存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合场景中，停止写入可能也意味着停止读取。 

### MyISAM 
设计简单，数据以紧密格式存储。对于只读数据，或者表比较小、可以容忍修复操作，则依然可以使用它。 

不支持事务。 
不支持行级锁，只能对整张表加锁，读取时会对需要读到的所有表加共享锁，写入时则对表加排它锁。但在表有读取操作的同时，也可以往表中插入新的记录，这被称为并发插入(CONCURRENT INSERT)。 

### 比较 
| 对比项    | MyISAM | InnoDB   |
| :---   | :--- | :---  |
| 主外键    | 不支持 |  支持    |
| 事务    | 不支持  |  支持 |
| 行表锁  | 表锁，即使操作一条记录也会锁住整个表，不适合高并发的操作  |  行锁,操作时只锁某一行，不对其它行有影响，适合高并发的操作     |
| 缓存  | 只缓存索引，不缓存真实数据  |  不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响     |
| 表空间    |  	小 |  小    |
| 关注点    |  	性能  |   	事务 |


## 索引
> 索引其实是一种数据结构，能够帮助我们快速的检索数据库中的数据，索引是在存储引擎层实现的，而不是在服务器层实现的。可以简单的理解为“排好序的快速查找数据结构”，数据本身之外，数据库还维护者一个满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。下图是一种可能的索引方式示例。

![](/images/db/rdb/index.jpg "索引")
### 索引类型
#### 数据结构角度
* B+树索引
* Hash索引
* Full-Text全文索引
* R-Tree索引

#### 从物理存储角度
* 聚集索引（clustered index）
* 非聚集索引（non-clustered index），也叫辅助索引（secondary index）
* 聚集索引和非聚集索引都是B+树结构

#### 从逻辑角度
* 主键索引：主键索引是一种特殊的唯一索引，不允许有空值
* 普通索引或者单列索引：每个索引只包含单个列，一个表可以有多个单列索引
* 多列索引（复合索引、联合索引）：复合索引指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用复合索引时遵循最左前缀集合
* 唯一索引或者非唯一索引
* 空间索引：空间索引是对空间数据类型的字段建立的索引，MYSQL中的空间数据类型有4种，分别是GEOMETRY、POINT、LINESTRING、POLYGON。
MYSQL使用SPATIAL关键字进行扩展，使得能够用于创建正规索引类型的语法创建空间索引。创建空间索引的列，必须将其声明为NOT NULL，空间索引只能在存储引擎为MYISAM的表中创建
MySQL主要有两种结构Hash索引和B+Tree索引，我们使用的是InnoDB引擎，默认的是B+树。

### B+Tree索引
> 所有的数据都存放在叶子节点上，且把叶子节点通过指针连接到一起，形成了一条数据链表，以加快相邻数据的检索效率。
##### 为什么使用B+树的数据结构
###### B-Tree
B-Tree 结构的数据可以让系统高效的找到数据所在的磁盘块。为了描述 B-Tree，首先定义一条记录为一个二元组[key, data] ，key为记录的键值，对应表中的主键值，data 为一行记录中除主键外的数据。对于不同的记录，key值互不相同。

B-Tree中的每个节点根据实际情况可以包含大量的关键字信息和分支，如下图所示为一个3阶的B-Tree:
![](/images/ds/b-tree.png  "B树")

###### B+Tree
B+Tree 是在 B-Tree 基础上的一种优化，使其更适合实现外存储索引结构，InnoDB 存储引擎就是用 B+Tree 实现其索引结构。

从上一节中的B-Tree结构图中可以看到每个节点中不仅包含数据的key值，还有data值。而每一个页的存储空间是有限的，如果data数据较大时将会导致每个节点（即一个页）能存储的key的数量很小，当存储的数据量很大时同样会导致B-Tree的深度较大，增大查询时的磁盘I/O次数，进而影响查询效率。

在B+Tree中，**所有数据记录节点都是按照键值大小顺序存放在同一层的叶子节点上，而非叶子节点上只存储key值信息，这样可以大大加大每个节点存储的key值数量，降低B+Tree的高度。**

B+Tree相对于B-Tree有几点不同：
* 非叶子节点只存储键值信息；
* 所有叶子节点之间都有一个链指针；
* 数据记录都存放在叶子节点中

将上一节中的B-Tree优化，由于B+Tree的非叶子节点只存储键值信息，假设每个磁盘块能存储4个键值及指针信息，则变成B+Tree后其结构如下图所示:
![](/images/ds/b+tree.png  "B+树")

因为不再需要进行全表扫描，只需要对树进行搜索即可，因此查找速度快很多。

除了用于查找，还可以用于排序和分组。 可以指定多个列作为索引列，多个索引列共同组成键。 

适用于全键值、键值范围和键前缀查找，其中键前缀查找只适用于最左前缀查找。如果不是按照索引列的顺序进行查找，则无法使用索引。 

InnoDB 的 B+Tree 索引分为主索引和辅助索引。 

Mysql选用B+树这种数据结构作为索引，可以提高查询索引时的磁盘IO效率，并且可以提高范围查询的效率，并且B+树里的元素也是有序的。

### 哈希索引
哈希索引能以 O(1) 时间进行查找，但是失去了有序性，它具有以下限制: 
* 无法用于排序与分组； 
* **只支持等值精确查找，无法用于部分查找和范围查找。**
* 哈希索引没办法利用索引完成排序 
* 哈希索引不支持多列联合索引的最左匹配规则 
* 如果有大量重复键值得情况下，哈希索引的效率会很低，因为存在哈希碰撞问题

InnoDB 存储引擎有一个特殊的功能叫“自适应哈希索引”，当某个索引值被使用的非常频繁时，会在 B+Tree 索引之上再创建一个哈希索引，这样就让 B+Tree 索引具有哈希索引的一些优点，比如快速的哈希查找。

### B+Tree的叶子节点存储内容不同分为聚簇索引和非聚簇索引

**主键索引的叶子节点 data 域记录着完整的数据记录，这种索引方式被称为聚簇索引。因为无法把数据行存放在两个不同的地方，所以一个表只能有一个聚簇索引。**
![](/images/db/rdb/PKIndex.jpg "聚簇索引")

**辅助索引的叶子节点的 data 域记录着主键的值，因此在使用辅助索引进行查找时，需要先查找到主键值，然后使用主键在主索引上再进行对应的检索操作，这也就是所谓的“回表查询”，如果覆盖索引则无需再“回表查询”**
![](/images/db/rdb/fuzhuIndex.jpg "非聚簇索引")

### 覆盖索引（covering index）
指一个查询语句的执行只用从索引中就能够取得，不必从数据表中读取。也可以称之为实现了索引覆盖。 

当一条查询语句符合覆盖索引条件时，MySQL只需要通过索引就可以返回查询所需要的数据，这样避免了查到索引后再返回表操作，减少I/O提高效率。 

如，表covering_index_sample中有一个普通索引 idx_key1_key2(key1,key2)。当我们通过SQL语句：select key2 from covering_index_sample where key1 = 'keytest';的时候，就可以通过覆盖索引查询，无需回表。

### 联合索引、最左前缀匹配
创建多列索引时，我们根据业务需求，where子句中使用最频繁的一列放在最左边，因为MySQL索引查询会遵循最左前缀匹配的原则，即最左优先，在检索数据时从联合索引的最左边开始匹配。

所以当我们创建一个联合索引的时候，如(key1,key2,key3)，相当于创建了（key1）、(key1,key2)和(key1,key2,key3)三个索引，这就是最左匹配原则。

### 索引下推
> MySQL 5.6引入了索引下推优化，默认开启，使用SET optimizer_switch = 'index_condition_pushdown=off';可以将其关闭。

**索引下推优化，可以在有like条件查询的情况下，减少回表次数。**

### 哪些情况需要创建索引
* 主键自动建立唯一索引
* 频繁作为查询条件的字段
* 查询中与其他表关联的字段，外键关系建立索引
* 单键/组合索引的选择问题，高并发下倾向创建组合索引
* 查询中排序的字段，排序字段通过索引访问大幅提高排序速度
* 查询中统计或分组字段

### 哪些情况不要创建索引
* 表记录太少
* 经常增删改的表
* 数据重复且分布均匀的表字段，只应该为最经常查询和最经常排序的数据列建立索引（如果某个数据类包含太多的重复数据，建立索引没有太大意义）
* 频繁更新的字段不适合创建索引（会加重IO负担）
* where条件里用不到的字段不创建索引

## 事务
> 事务指的是满足 ACID 特性的一组操作，可以通过 Commit 提交一个事务，也可以使用 Rollback 进行回滚。
### ACID特性
#### 原子性(Atomicity) 
事务被视为不可分割的最小单元，事务的所有操作要么全部提交成功，要么全部失败回滚。 回滚可以用日志来实现，日志记录着事务所执行的修改操作，在回滚时反向执行这些修改操作即可。
##### 一致性(Consistency) 
数据库在事务执行前后都保持一致性状态。在一致性状态下，所有事务对一个数据的读取结果都是相同的。 
#### 隔离性(Isolation) 
一个事务所做的修改在最终提交以前，对其它事务是不可见的。
#### 持久性(Durability) 
一旦事务提交，则其所做的修改将会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢失。 可以通过数据库备份和恢复来实现，在系统发生崩溃时，使用备份的数据库进行数据恢复。

#### ACID特性的相互关系
事务的 ACID 特性概念简单，但不是很好理解，主要是因为这几个特性不是一种平级关系: 
* 只有满足一致性，事务的执行结果才是正确的。 
* 在无并发的情况下，事务串行执行，隔离性一定能够满足。此时只要能满足原子性，就一定能满足一致性。 
* 在并发的情况下，多个事务并行执行，事务不仅要满足原子性，还需要满足隔离性，才能满足一致性。 
* 事务满足持久化是为了能应对数据库崩溃的情况。

![](/images/db/rdb/dbc.png "ACID")

### 并发一致性问题
在并发环境下，**事务的隔离性很难保证，因此会出现很多并发一致性问题。**
#### 丢失修改
T1和T2两个事务都对一个数据进行修改，T1先修改，T2随后修改，**T2的修改覆盖了T1的修改**。
#### 读脏数据
T1修改一个数据，T2随后读取这个数据。如果**T1撤销了这次修改，那么T2读取的数据是脏数据。**
#### 不可重复读
**T2读取一个数据，T1对该数据做了修改**。如果T2再次读取这个数据，此时读取的结果和第一次读取的结果不同。
#### 幻影读
**T1读取某个范围的数据，T2在这个范围内插入新的数据，T1再次读取这个范围的数据**，此时读取的结果和和第一次读取的结果不同。
#### 幻读和不可重复读的区别：
* 不可重复读的重点是修改：在同一事务中，同样的条件，第一次读的数据和第二次读的数据不一样。（因为中间有其他事务提交了修改）
* 幻读的重点在于新增或者删除：在同一事务中，同样的条件，第一次和第二次读出来的记录数不一样。（因为中间有其他事务提交了插入/删除）

### 并发一致性的解决方案（引出隔离级别等）
**产生并发不一致性问题主要原因是破坏了事务的隔离性，解决方法是通过并发控制来保证隔离性**。并发控制可以通过封锁来实现，但是**封锁操作**需要用户自己控制，相当复杂。

**数据库管理系统提供了事务的隔离级别，让用户以一种更轻松的方式处理并发一致性问题。**

### 隔离级别 
> 使用select @@tx_isolation;查询数据库的隔离级别。Mysql默认的级别是可重复读，优先考虑把数据库系统的隔离级别设为读已提交。

#### 未提交读(READ UNCOMMITTED) 
读未提交，一个事务可以读到另一个事务未提交的数据！
#### 提交读(READ COMMITTED) 
读已提交，一个事务可以读到另一个事务已提交的数据!
一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其它事务是不可见的。 
#### 可重复读(REPEATABLE READ) 
可重复读，加入间隙锁保证在同一个事务中多次读取同样数据的结果是一样的。
#### 可串行化(SERIALIZABLE) 
串行化，该级别下读写串行化，且所有的select语句后都自动加上lock in share mode，即使用了共享锁。因此在该隔离级别下，使用的是当前读，而不是快照读。

**多版本并发控制是MySQL的InnoDB存储引擎实现隔离级别的一种具体方式**，用于实现**提交读和可重复读这两种隔离级别**。

**而未提交读隔离级别总是读取最新的数据行，无需使用 MVCC。可串行化隔离级别需要对所有读取的行都加锁，单纯使用 MVCC 无法实现。**

MySQL的InnoDB存储引擎**采用两段锁协议，会根据隔离级别在需要的时候自动加锁，并且所有的锁都是在同一时刻被释放**，这被称为隐式锁定。

![](/images/db/rdb/geli.png "隔离级别")

### 多版本并发控制
> 多版本并发控制(Multi-Version Concurrency Control, MVCC)是 MySQL 的 InnoDB 存储引擎实现隔离级别的一种具体方式，用于实现提交读和可重复读这两种隔离级别。而未提交读隔离级别总是读取最新的数据行，无需使用 MVCC。可串行化隔离级别需要对所有读取的行都加锁，单纯使用 MVCC 无法实现。

#### 版本号 
* 系统版本号: 是一个递增的数字，每开始一个新的事务，系统版本号就会自动递增。 
* 事务版本号: 事务开始时的系统版本号。 

#### 隐藏的列 
MVCC 在每行记录后面都保存着两个隐藏的列，用来存储两个版本号: 
* 创建版本号: 指示创建一个数据行的快照时的系统版本号； 
* 删除版本号: 如果该快照的删除版本号大于当前事务版本号表示该快照有效，否则表示该快照已经被删除了。 

#### Undo 日志 
MVCC 使用到的快照存储在 Undo 日志中，该日志通过回滚指针把一个数据行(Record)的所有快照连接起来。

#### 实现过程
以下实现过程针对可重复读隔离级别。

当开始新一个事务时，该事务的版本号肯定会大于当前所有数据行快照的创建版本号，理解这一点很关键。

1. SELECT 
> 多个事务必须读取到同一个数据行的快照，并且这个快照是距离现在最近的一个有效快照。但是也有例外，如果有一个事务正在修改该数据行，那么它可以读取事务本身所做的修改，而不用和其它事务的读取结果一致。 把没有对一个数据行做修改的事务称为 T，T 所要读取的数据行快照的创建版本号必须小于 T 的版本号，因为如果大于或者等于 T 的版本号，那么表示该数据行快照是其它事务的最新修改，因此不能去读取它。除此之外，T 所要读取的数据行快照的删除版本号必须大于 T 的版本号，因为如果小于等于 T 的版本号，那么表示该数据行快照是已经被删除的，不应该去读取它。 

2. INSERT 
> 将当前系统版本号作为数据行快照的创建版本号。 

3. DELETE 
> 将当前系统版本号作为数据行快照的删除版本号。 

4. UPDATE 
> 将当前系统版本号作为更新前的数据行快照的删除版本号，并将当前系统版本号作为更新后的数据行快照的创建版本号。可以理解为先执行 DELETE 后执行 INSERT。

#### 快照读与当前读
##### 快照读
使用 MVCC 读取的是快照中的数据，这样可以减少加锁所带来的开销。
```sql
select * from table ...;
```
##### 当前读
读取的是最新的数据，需要加锁。以下第一个语句需要加 S 锁，其它都需要加 X 锁。
```sql
select * from table where ? lock in share mode; //S锁 (共享锁)
select * from table where ? for update;         //加X锁 (排他锁)
insert;
update;
delete;
```

### 事务的实现
事务的实现是基于数据库的存储引擎。不同的存储引擎对事务的支持程度不一样。MySQL 中支持事务的存储引擎有 InnoDB 和 NDB。

事务的实现就是如何实现ACID特性。**事务的隔离性是通过锁实现，而事务的原子性、一致性和持久性则是通过事务日志实现。**

nnodb事务日志包括redo log和undo log。
#### redo log（重做日志）
redo log通常是物理日志，记录的是数据页的物理修改，而不是某一行或某几行修改成怎样，它用来恢复提交后的物理数据页。

在innoDB的存储引擎中，事务日志通过重做(redo)日志和innoDB存储引擎的日志缓冲(InnoDB Log Buffer)实现。事务开启时，事务中的操作，都会先写入存储引擎的日志缓冲中，在事务提交之前，这些缓冲的日志都需要提前刷新到磁盘上持久化，这就是DBA们口中常说的“日志先行”(Write-Ahead Logging)。当事务提交之后，在Buffer Pool中映射的数据文件才会慢慢刷新到磁盘。此时如果数据库崩溃或者宕机，那么当系统重启进行恢复时，就可以根据redo log中记录的日志，把数据库恢复到崩溃前的一个状态。未完成的事务，可以继续提交，也可以选择回滚，这基于恢复的策略而定。

在系统启动的时候，就已经为redo log分配了一块连续的存储空间，以顺序追加的方式记录Redo Log，通过顺序IO来改善性能。所有的事务共享redo log的存储空间，它们的Redo Log按语句的执行顺序，依次交替的记录在一起。

#### undo log（回滚日志）
undo log是逻辑日志，和redo log记录物理日志的不一样。可以这样认为，当delete一条记录时，undo log中会记录一条对应的insert记录，当update一条记录时，它记录一条对应相反的update记录。

undo log 主要为事务的回滚服务。在事务执行的过程中，除了记录redo log，还会记录一定量的undo log。undo log记录了数据在每个操作前的状态，如果事务执行过程中需要回滚，就可以根据undo log进行回滚操作。单个事务的回滚，只会回滚当前事务做的操作，并不会影响到其他的事务做的操作。

Undo记录的是已部分完成并且写入硬盘的未完成的事务，默认情况下回滚日志是记录下表空间中的（共享表空间或者独享表空间）

二种日志均可以视为一种恢复操作，redo_log是恢复提交事务修改的页操作，而undo_log是回滚行记录到特定版本。二者记录的内容也不同，redo_log是物理日志，记录页的物理修改操作，而undo_log是逻辑日志，根据每行记录进行记录。

#### 事务ACID特性的实现思想
* 原子性：是使用 undo log来实现的，如果事务执行过程中出错或者用户执行了rollback，系统通过undo log日志返回事务开始的状态。
* 持久性：使用 redo log来实现，只要redo log日志持久化了，当系统崩溃，即可通过redo log把数据恢复。
* 隔离性：通过锁以及MVCC,使事务相互隔离开。
* 一致性：通过回滚日志、恢复，以及并发情况下的隔离性，从而实现一致性。

### MySQL 有多少种日志
* 错误日志：记录出错信息，也记录一些警告信息或者正确的信息。
* 查询日志：记录所有对数据库请求的信息，不论这些请求是否得到了正确的执行。
* 慢查询日志：设置一个阈值，将运行时间超过该值的所有SQL语句都记录到慢查询的日志文件中。
* 二进制日志：记录对数据库执行更改的所有操作。
* 中继日志：中继日志也是二进制日志，用来给slave 库恢复
* 事务日志：重做日志redo和回滚日志undo

### MySQL对分布式事务的支持
>下面是简单描述，详细请参考分布式事务。

分布式事务的实现方式有很多，既可以采用 InnoDB 提供的原生的事务支持，也可以采用消息队列来实现分布式事务的最终一致性。这里我们主要聊一下 InnoDB 对分布式事务的支持。

MySQL 从 5.0.3  InnoDB 存储引擎开始支持XA协议的分布式事务。一个分布式事务会涉及多个行动，这些行动本身是事务性的。所有行动都必须一起成功完成，或者一起被回滚。

## MySQL锁机制
### 封锁
#### 封锁粒度
MySQL 中提供了两种封锁粒度: 行级锁以及表级锁。
![](/images/db/rdb/dblock.jpg "封锁粒度")

#### 封锁类型
##### 读写锁
* 排它锁(Exclusive)，简写为 X 锁，又称写锁。 
* 共享锁(Shared)，简写为 S 锁，又称读锁。 

有以下两个规定: 
* 一个事务对数据对象 A 加了 X 锁，就可以对 A 进行读取和更新。加锁期间其它事务不能对 A 加任何锁。 
* 一个事务对数据对象 A 加了 S 锁，可以对 A 进行读取操作，但是不能进行更新操作。加锁期间其它事务能对 A 加 S 锁，但是不能加 X 锁。

##### 意向锁
使用意向锁(Intention Locks)可以更容易地支持多粒度封锁。 

在存在行级锁和表级锁的情况下，事务T想要对表A加X锁，就需要先检测是否有其它事务对表A或者表A中的任意一行加了锁，那么就需要**对表A的每一行都检测一次，这是非常耗时的**。

意向锁在原来的 X/S 锁之上引入了 IX/IS，IX/IS 都是表锁，用来表示一个事务想要在表中的某个数据行上加 X 锁或 S 锁。

* 一个事务在获得某个数据行对象的 S 锁之前，必须先获得表的 IS 锁或者更强的锁； 
* 一个事务在获得某个数据行对象的 X 锁之前，必须先获得表的 IX 锁。 
* 任意 IS/IX 锁之间都是兼容的，因为它们只是表示想要对表加锁，而不是真正加锁；
* S 锁只与 S 锁和 IS 锁兼容，也就是说事务 T 想要对数据行加 S 锁，其它事务可以已经获得对表或者表中的行的 S 锁。

**通过引入意向锁，事务 T 想要对表 A 加 X 锁，只需要先检测是否有其它事务对表A加了 X/IX/S/IS 锁，如果加了就表示有其它事务正在使用这个表或者表中某一行的锁，因此事务T加X锁失败。**

#### 封锁协议
##### 三级封锁协议
###### 一级封锁协议
事务 T 要修改数据 A 时必须加 X 锁，直到 T 结束才释放锁。

可以解决丢失修改问题，因为不能同时有两个事务对同一个数据进行修改，那么事务的修改就不会被覆盖。
###### 二级封锁协议 
在一级的基础上，要求读取数据 A 时必须加 S 锁，读取完马上释放 S 锁。 

可以解决读脏数据问题，因为如果一个事务在对数据 A 进行修改，根据 1 级封锁协议，会加 X 锁，那么就不能再加 S 锁了，也就是不会读入数据。
###### 三级封锁协议 
在二级的基础上，要求读取数据 A 时必须加 S 锁，直到事务结束了才能释放 S 锁。 

可以解决不可重复读的问题，因为读 A 时，其它事务不能对 A 加 X 锁，从而避免了在读的期间数据发生改变。

##### 两段锁协议
加锁和解锁分为两个阶段进行。 

可串行化调度是指，通过并发控制，使得并发执行的事务结果与某个串行执行的事务结果相同。 

事务遵循两段锁协议是保证可串行化调度的充分条件。例如以下操作满足两段锁协议，它是可串行化调度。

MySQL 的 InnoDB 存储引擎采用两段锁协议，会根据隔离级别在需要的时候自动加锁，并且所有的锁都是在同一时刻被释放，这被称为隐式锁定。

### 锁模式(InnoDB有三种行锁的算法)
#### Record Locks（记录锁，行锁）
单个行记录上的锁。对索引项加锁，锁定符合条件的行。其他事务不能修改和删除加锁项。

该锁是对索引记录进行加锁！锁是在加索引上而不是行上的。注意了，innodb一定存在聚簇索引，因此行锁最终都会落到聚簇索引上！

如果表没有设置索引，InnoDB 会自动在主键上创建隐藏的聚簇索引，因此 Record Locks 依然可以使用。 

#### Gap Locks（间隙锁）
> 当我们使用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁。对于键值在条件范围内但并不存在的记录，叫做“间隙”，InnoDB 也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁。

锁定索引之间的间隙，但是不包含索引本身。其目的只有一个，防止其他事物插入数据。

**间隙锁基于非唯一索引，它锁定一段范围内的索引记录。间隙锁基于下面将会提到的Next-Key Locking 算法，请务必牢记：使用间隙锁锁住的是一个区间，而不仅仅是这个区间中的每一条数据。**

隔离级别比Read Committed低的情况下，不会使用间隙锁，如隔离级别为Read Uncommited时，也不存在间隙锁。当隔离级别为Repeatable Read和Serializable时，就会存在间隙锁。

例如当一个事务执行以下语句，其它事务就不能在 t.c 中插入 15。
```sql
SELECT c FROM t WHERE c BETWEEN 10 and 20 FOR UPDATE;
```

#### Next-Key Locks（临键锁）
临键锁，是记录锁与间隙锁的组合，它的封锁范围，既包含索引记录，又包含索引区间。

MVCC不能解决幻读的问题，Next-Key Locks 就是为了解决这个问题而存在的。**在可重复读(REPEATABLE READ)隔离级别下，使用 MVCC + Next-Key Locks 可以解决幻读问题。**

例如一个索引包含以下值: 10, 11, 13, and 20，那么就需要锁定以下区间:
```sql
(negative infinity, 10]
(10, 11]
(11, 13]
(13, 20]
(20, positive infinity)
```
### 锁行锁表
InnoDB行锁是通过给索引上的索引项加锁来实现，只有通过索引条件检索数据，InnoDB才使用行级锁，否则表锁（注意下面的解释）。

这里的表锁并不是用表锁来实现锁表的操作，而是利用了Next-Key Locks，也可以理解为是用了行锁+间隙锁来实现锁表的操作! 

之所以能够锁表，是通过行锁+间隙锁来实现的。那么，RU和RC都不存在间隙锁，这种说法在RU和RC中还能成立么？ 因此，该说法只在RR和Serializable中是成立的。
**如果隔离级别为RU和RC，无论条件列上是否有索引，都不会锁表，只锁行！**

## 参考文章
* [MySQL总结](https://juejin.cn/post/6850037271233331208 "MySQL总结")
* [数据库系统核心知识点](https://www.pdai.tech/md/db/sql/sql-db-theory.html "数据库系统核心知识点")
* [MySQL索引数据结构详解](https://juejin.cn/post/6844904084886011911 "MySQL索引数据结构详解")
* [select加锁分析](https://juejin.cn/post/6844903919387148296 "select加锁分析")
* [MySQL InnoDB的MVCC实现机制](https://www.pdai.tech/md/db/sql-mysql/sql-mysql-mvcc.html "MySQL InnoDB的MVCC实现机制")
* [一条SQL的执行过程详解](https://www.pdai.tech/md/db/sql-mysql/sql-mysql-execute.html "一条SQL的执行过程详解")