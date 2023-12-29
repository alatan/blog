
---
title: "MySQL常用命令"
description: "MySQL总结"
keywords: ["MySQL"]
date: 2017-03-09
author: "默哥"
weight: 70
draft: false

categories: ["MySQL"]
tags: ["MySQL"]  
toc: 
    auto: false
---

## 登录
mysql -u root -p

## 远程登录
### 修改表
```sql
1. select user,host from user;
2. update user set host = '%' where user = 'root';
3. flush privileges;
```

### 权限方法
```sql
1. GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '您的数据库密码' WITH GRANT OPTION;
2. flush privileges;
```

## 查看日志相关信息
show variables like 'log_%';

## 慢查询日志：slow query log
show global status like '%Slow_queries%';
show variables like '%slow_query%';
show variables like 'long_query_time%';

## 查看MySQL数据文件
show variables like '%datadir%';

## 性能瓶颈定位

我们可以通过 show 命令查看 MySQL 状态及变量，找到系统的瓶颈：

```sql
Mysql> show status ——显示状态信息（扩展show status like ‘XXX’）
Mysql> show variables ——显示系统变量（扩展show variables like ‘XXX’）
Mysql> show innodb status ——显示InnoDB存储引擎的状态
Mysql> show processlist ——查看当前SQL执行，包括执行状态、是否锁表等
Shell> mysqladmin variables -u username -p password——显示系统变量
Shell> mysqladmin extended-status -u username -p password——显示状态信息
```