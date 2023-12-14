
---  
title: "MySQL常用命令"
description: "MySQL总结"
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