---
title: "MySQL存储引擎"
description: "MySQL存储引擎"
keywords: ["MySQL"]
date: 2017-03-02
author: "默哥"
weight: 70
draft: false

categories: ["MySQL"]
tags: ["MySQL"]  
toc: 
    auto: false
---

## 存储引擎对比
### 存储引擎种类
* InnoDB
* MyISAM
* Memory
* Falcon
### InnoDB VS MyISAM
![](/images/db/rdb/engineCompare.png "存储引擎对比")


## 存储引擎之InnoDB
![](/images/db/rdb/InnoDB-struct.png " InnoDB架构图")

上图详细展示了InnoDB存储引擎的体系架构，从图中可见，InnoDB存储引擎由**内存结构、磁盘结构**两部分组成。

## 内存结构
InnoDB 内存结构主要分为如下四个区域：
1. Buffer Pool 缓冲池
2. Change Buffer 修改缓冲
3. Adaptive Hash Index 自适应索引
4. Log Buffer 日志缓冲

### 缓冲池Buffer Pool
缓冲池Buffer Pool 用于加速数据的访问和修改，通过将热点数据缓存在内存的方法，最大限度地减少磁盘 IO，加速热点数据读写。
* 默认大小为128M，Buffer Pool 中数据以**页为存储单位**，其实现的数据结构是**以页为单位的链表**。
* 缓存数据到内存，最大限度地减少磁盘 IO，加速热点数据的读和写
* 由于内存有限，Buffer Pool 仅能容纳最热点的数据
* 使用LRU算法（最近最少使用）淘汰非热点数据页：
    * LRU：根据页数据的历史访问来淘汰数据，**如果数据最近被访问过，那么将来被访问的几率也更高**，优先淘汰最近没有被访问到的数据。
* 对于 Buffer Pool 中数据的查询，InnoDB 直接读取返回。对于 Buffer Pool 中数据的修改，InnoDB 直接在 Buffer Pool 中修改，并将修改写入redo log。

![](/images/db/rdb/LRU.png " LRU算法")

![](/images/db/rdb/buffer-pool-data.png "以页为单位的链表")

```sql
# 查看innodb存储引擎状态，包含缓冲池、修改缓冲、自适应哈希状态信息、日志缓冲等信息...
mysql> show engine innodb status;
# 查看InnoDB的Buffer Pool大小
mysql> show variables like 'innodb_buffer_pool_size';
```

### 修改缓冲Change Buffer
* 用于加速非热点数据中二级索引的写入操作
* 修改缓冲对二级索引的修改操作会录入 redo log 中
* 在缓冲到一定量或系统较空闲时进行merge操作（写入磁盘）
* 修改缓冲在系统表空间中有相应的持久化区域
* Change Buffer 大小默认占 Buffer Pool 的 25%，最大50%，在引擎启动时便初始化完成。其物理结构为一棵名为 ibuf 的 B Tree。


* **二级索引**就是辅助索引，除了聚簇索引之外的所有索引都是二级索引。
* **聚簇索引**也叫聚集索引，索引组织表，指的**是一种数据存储方式，指数据与索引的数据结构存储在一起**。如 InnoDB 的主键索引中所有叶子节点都存储了对应行的数据。因为数据肯定只是存储在一个地方，所以一个表只能有一个聚集索引。

![](/images/db/rdb/Change-Buffer.png "修改缓冲")

### 自适应哈希索引Adaptive Hash Index
* 自适应哈希索引（Adaptive Hash Index，AHI）用于实现对于热数据页的一次查询，**是建立在索引之上的索引**
    * 使用聚簇索引进行数据页定位的时候需要根据索引树的高度从根节点走到叶子节点，通常需要3 到 4 次查询才能定位到数据。InnoDB 根据对索引使用情况的分析和索引字段的分析，通过自调优Self-tuning的方式为索引页建立或者删除哈希索引
* **作用：对频繁查询的数据页和索引页进一步提速**
* AHI 的大小为 Buffer Pool 的 1/64，在 MySQL 5.7 之后支持分区，以减少对于全局 AHI 锁的竞争，默认分区数为 8。
* **若二级索引命中AHI**：
    * **从AHI获取索引页记录指针，再根据主键沿着聚簇索引查找数据**
* **若聚簇索引命中AHI**
    * **则直接返回目标数据页的记录指针，根据记录指针可以直接定位数据页**

![](/images/db/rdb/AHI.png "自适应哈希索引")

### 日志缓冲(Log Buffer)
* InnoDB 使用 Log Buffer 来缓冲日志文件的写入操作
* **内存写入**加上**日志文件顺序写**使得 InnoDB 日志写入性能极高。
* Log Buffer将分散的写入操作放在内存中，通过定期批量写入磁盘的方式提高日志写入效率和减少磁盘 IO。

![](/images/db/rdb/Log-Buffer.png "日志缓冲")

## 磁盘结构
在磁盘中，InnoDB存储引擎将数据、索引、表结构和其他缓存信息等存放的空间称为表空间（Tablespace）它是物理存储中的最高层，由段Segment、区extent、页Page、行Row组成。
* 开启独立表空间innodb_file_per_table=1，每张表的数据都会存储到一个独立表空间，即 表名.ibd 文件
* 关闭独占表空间innodb_file_per_table=0，则所有基于InnoDB存储引擎的表数据都会记录到系统表空间，即 ibdata1 文件

### 表空间是 InnoDB 物理存储中的最高层，目前的表空间类别包括：
* 系统表空间（System Tablespace），关闭独立表空间，所有表数据和索引都会存入系统表空间
* 独立表空间（File-per-table Tablespace），开启独立表空间，每张表的数据都会存储到一个独立表空间
* 通用表空间（General Tablespace）
* 回滚表空间（Undo Tablespace）
* 临时表空间（The Temporary Tablespace）

![](/images/db/rdb/Tablespace.png "表空间")

### 系统表空间（System Tablespace）
*  **系统表空间是 InnoDB 数据字典、双写缓冲、修改缓冲和回滚日志的存储位置**如果关闭独立表空间，它也将存储所有表数据和索引。
    * 数据字典：表对象的元数据信息（表结构、索引、列信息等等）
    * **双写缓冲**：保证写入数据时，页数据的完整性，防止部分写失效问题
    * **修改缓冲**：内存中Change Buffer对应的持久化区
    * **回滚日志**：实现数据**回滚**时对数据的恢复，MVCC实现的重要组成
*  默认大小12M，文件名称ibdata1，配置参数：innodb_data_file_path
*  系统表空间会自动增长，每次增量64MB，且增长之后的文件不可缩减。

### 独立表空间（File-per-table Tablespace）
* **独立表空间用于存放每个表的数据和索引。在5.7版本中默认开启，初始化大小是96KB**
* 其他类型的信息，如：回滚日志、双写缓冲区、系统事务信息、修改缓冲等仍存放于系统表空间内，因此即使用了独立表空间，系统表空间也会不断增长。
* 开启关闭： innodb_file_per_table=ON|OFF
* 在数据库文件夹内，为每张表单独创建表空间 表名.ibd 文件存数据与索引。同时创建 表名.frm 文件存表结构信息

### 通用表空间（General Tablespace）
* 通用表空间存在的目的是为了在系统表空间与独立表空间之间作出平衡
* 系统表空间与独立表空间中的表可以向通用表空间移动，反之亦可
* 但系统表空间中的表无法直接与独立表空间中的表相互转化。
![](/images/db/rdb/General-Tablespace.png "通用表空间")

### 回滚表空间（Undo Tablespace）
* Undo TableSpace 用于存放一个或多个 undo log 文件，默认大小为10MB
* Undo log默认存储在系统表空间，在5.7版本中支持自定义到独立的表空间

### 临时表空间（The Temporary Tablespace）
* MySQL 5.7 之前临时表存储在系统表空间，这样会导致ibdata在使用临时表的场景下疯狂增长
* MySQL 5.7 版本里InnoDB引擎从系统表空间中抽离出临时表空间独立保存临时表数据
* 配置属性： innodb_temp_data_file_path

## 磁盘结构-存储结构
![](/images/db/rdb/tableSpaceStoage.png "磁盘结构-存储结构")

表空间由段（Segment）、区（extent）、页（Page）、行（Row）组成

### 段（Segment）
* 表空间由各个段组成，段类型分数据段、索引段、回滚段。
* MySQL的索引数据结构是B+树，这个树有叶子节点和非叶子节点
* 一个段包含多个区，至少有一个区，段扩展的最小单位是区
    * 数据段是叶子节点 Leaf node segment
    * 索引段是非叶子节点Non-Leaf node segment
### 区（Extent）
* 区是由连续的页组成的空间，大小固定为 1MB
* 默认情况下，一个区里有64个页
* 为了保证区的连续性，InnoDB一次会从磁盘申请4-5个区
### 页（Page）
* 页是 InnoDB 的基本存储单位，页默认大小是16K（可配置innodb_page_size），InnoDB 首次加载后便无法更改
* 操作系统读写磁盘最小单位是页，4K
* 磁盘存储数据量最小单位512 byte

![](/images/db/rdb/db-page.png "页大小")

### 行（Row）
* InnoDB的数据是以行为单位存储，一个页中包含多个行
* InnoDB提供4种行格式：Compact、Redundant、Dynamic和Compressed
* 默认行格式Dynamic

## 内存中的数据如何进入磁盘
![](/images/db/rdb/dataStroage.png "内存数据落盘")

* 在数据库中进行读取操作，将从磁盘中读到的页放在缓冲区中，下次再读相同的页时，首先判断该页是否在缓冲区中。若在缓冲区中，称该页在缓冲区中被命中，直接读取该页。否则读取磁盘上的页。

* 对于数据库中页的修改操作，则首先修改在缓冲区中的页，然后再以一定的频率刷新到磁盘上。页从缓冲区刷新回磁盘的操作并不是在每次页发生更新时都触发，而是通过一种称为CheckPoint的机制刷新回磁盘。

### 数据库灵魂三问：
* **写入性能如何保证**？
    * 分散写入操作放在内存中，通过定期批量写入磁盘的方式提高写入效率减少磁盘 IO。
* **如何持久化**？也就是修改后的数据如何到磁盘中去。内存里缓冲池中的数据页要完成持久化通过两个流程来完成
    * 通过CheckPoint机制进行**脏页落盘**
    * **日志先行**，所有操作前先写Redo日志
* **数据安全性怎么保证**？
    * 记录操作日志：**Force Log at Commit机制与Write Ahead Log（WAL）策略**
    * CheckPoint机制
    * Double Write机制

### 脏页落盘
![](/images/db/rdb/InnoDB-struct.png "InnoDB架构图")

### 什么是脏页？
* 对数据的**修改操作**，首先修改**内存结构**中**缓冲区**中的页，**缓冲区**中的页与**磁盘**中的页数据不一致，所以称缓冲区中的页为**脏页**。
### 脏页如何进入到磁盘？
* 脏页从缓冲区刷新到磁盘，不是每次页更新之后触发，而是通过**CheckPoint机制**刷新磁盘
* 脏页落盘涉及的问题很多，咱们会重点分析，首先咱们来看下数据整体落盘流程

### 如何提升性能？
* **内存中写**，操作过程**记录日志**，日志批量写入磁盘
* 分散写变顺序写
### 如何持久化数据？
* 通过CheckPoint机制进行**脏页落盘**
* **日志先行**，所有操作前先写Redo日志

### 数据安全性怎么保证？
* Force Log at Commit
* Write Ahead Log（WAL）
* CheckPoint机制
* Double Write机制
### 为什么不是每次更新直接写入磁盘？
* 如果每次页发生变化就落盘，一个页落盘必然伴随4次IO操作，性能开销很大，而且随着写入操作增加性能开销是指数级增长
* 当然，数据也不能在内存中保存太长时间，**时间越久安全性风险越高**
* InnoDB采用**Write Ahead Log**策略和Force Log at Commit机制实现事务的持久性
    * **Write Ahead Log：日志先行**，数据变更写入磁盘前，必须将内存中的日志写入到磁盘
    * **Force Log at Commit**：当事务提交时，所有事务产生的日志都必须刷到磁盘

![](/images/db/rdb/datalogstorage.png "写入")

### 怎么确保日志就能安全的写入系统呢？
* 为了确保日志写入到磁盘，将redo日志写入Log Buffer后调用fsync函数，将缓冲日志文件从OS Cache中写入磁盘
* 日志进入磁盘不仅要考虑安全性，还需兼顾性能！
### 这样做不就等同于数据直接写入磁盘吗
* redo日志不会记录完整的一页数据，因为这样日志太大，它只会记录那次（sequence）如何操作了（update,insert）哪页(page)的哪行(row)
* 日志是顺序写入，而数据是随机写入。顺序写入效率更高
* 日志也不是改一条写一条，而是采用redo 日志落盘策略来兼顾安全性与性能！
* 可以通过 innodb_flush_log_at_trx_commit 来控制redo日志刷新到磁盘的策略。

## Redo日志落盘
![](/images/db/rdb/redologStorage.png "Redo日志落盘")

**Redo日志默认落盘策略，事务提交立即落盘**。Log Buffer写入磁盘的时机由参数innodb_flush_log_at_trx_commit 控制，此参数控制每次事务提交时InnoDB的行为。
三个配置：
* **为0时**：每秒写入，与事务无关
    * 最多丢失1秒的事务操作
    * 写入效率最高，安全性最低
* **为1时**：事务提交，写入磁盘
    * 不会丢失数据
    * 写入效率最低，安全性最高
* **为2时**：事务提交，写入OS Buffer
    * 数据安全性依赖于系统，最多丢1秒事务操作
    * 写入效率居中，安全性居中

## CheckPoint机制
### 什么是CheckPoint机制？
* 它是将缓冲池中的脏页数据刷到磁盘上的机制，决定脏页落盘的时机、条件和脏页的选择等
* CheckPoint的类型不止一种
### 解决什么问题？
* **脏页落盘**：避免数据更改直接操作磁盘
* **缩短数据库的恢复时间：数据库宕机时**，不用重做所有redo日志，大大缩短恢复时间
* **缓冲池不够用时，将脏页刷新到磁盘**：Buffer Pool不够用时溢出页落盘，LRU淘汰掉的非热点数据
* **redo日志不可用时，刷新脏页**：日志文件可以循环使用，不会无限增长
### 分类
* sharp checkpoint：关闭数据库时将脏页全部刷新到磁盘中
* fuzzy checkpoint：默认方式，在运行时选择不同时机将脏页刷盘。只刷新部分脏页
    * Master Thread Checkpoint：固定频率刷新部分脏页到磁盘，异步操作不会阻塞用户线程
    * FLUSH_LRU_LIST Checkpoint：缓冲池淘汰非热点Page，如果该Page是脏页会执行CheckPoint
    * Async/Sync Flush Checkpoint：redo日志不可用时，强制脏页落盘，有了前两个这种一般不会发生
    * Dirty Page too much Checkpoint：脏页占比太多强制进行刷盘，阈值75%

## Double Write机制
### 写失效问题
数据库准备刷新脏页时，将16KB的刷入磁盘，但当写入了8KB时，就宕机了这种只写了部分没完成的情况被称为写**失效Partial Page Write**

![](/images/db/rdb/dubleWrite.png "写失效问题")

### Double Write机制
* Double Write其实就是写两次，在修改记录redo日志前，先做个副本留个“备胎”

**注意：Redo日志不能解决写失效问题，因为redo日志记录的是对页的修改记录而不是数据本身**

### 两个部分
* 内存中，大小为2MB
* 磁盘系统表空间中，也为2MB，2个区，连续128个页

### 流程
1. 首先复制
2. 在顺序写入磁盘第一次
3. 最后离散写入磁盘第二次

![](/images/db/rdb/dubleWriteStroage.png "Double Write写入机制")

### Double Write奔溃恢复过程
1. 首先找到系统表空间中Double Write区域对应的页副本数据
2. 然后将其复制到独立表空间
3. 最后清楚redo日志

![](/images/db/rdb/DoubleWriteHF.png "Double Write奔溃恢复过程")