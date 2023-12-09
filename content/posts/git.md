---  
title: "git 代理配置"  
date:  2017-01-01
weight: 70  
draft: false  
keywords: [""]  
description: "JVM-一个对象的一生（出生、死亡与内涵）"  
tags: ["git"]
categories: ["Tools"]
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

## 设置https 代理
* Git代理有两种设置方式，分别是全局代理和只对Github代理，建议只对github 代理。
* 代理协议也有两种，分别是使用http代理和使用socks5代理，建议使用socks5代理。
* 注意下面代码的端口号需要根据你自己的代理端口设定，比如我的代理socks端口是51837。

### 全局设置（不推荐）

```java
#使用http代理 
git config --global http.proxy http://127.0.0.1:58591
git config --global https.proxy https://127.0.0.1:58591
#使用socks5代理
git config --global http.proxy socks5://127.0.0.1:51837
git config --global https.proxy socks5://127.0.0.1:51837
```

### 只对Github代理（推荐）
```java
#使用socks5代理（推荐）
git config --global http.https://github.com.proxy socks5://127.0.0.1:51837
#使用http代理（不推荐）
git config --global http.https://github.com.proxy http://127.0.0.1:58591
```

### 取消代理
当你不需要使用代理时，可以取消之前设置的代理。
```java
git config --global --unset http.proxy git config --global --unset https.proxy  
```

## 参考文章
1.https://zhuanlan.zhihu.com/p/481574024