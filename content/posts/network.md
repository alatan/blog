--- 
title: "计算机网络"
description: "计算机网络"
keywords: ["网络"]
date: 2017-01-05
author: "默哥"
weight: 70
draft: false

categories: ["网络编程"]
tags: ["网络", "大纲"] 
toc: 
  auto: false
---
## 网络通信协议
* 是指一种网络通用语言，为连接不同操作系统和不同硬件体系结构的互联网络提供通信支持的一堆协议
* 常见的网络通信协议： TCP/IP协议、IPX/SPX协议、NetBEUI协议等。
* 主要是对信息传输的速率、传输代码、代码结构、传输控制步骤、出错控制等作出规定并制定出标准。

### TCP/IP协议
* 互联网相关联的协议集的总称：TCP/IP
* 全称传输控制协议/因特网互联协议( Transmission Control Protocol/Internet Protocol)
* 是最基本、应用最广泛的一套互联网协议，定义了计算机如何接入互联网，数据在互联网如何传输
* TCP/IP四层体系结构： 针对应用开发者的OSI体系结构简化版
  * OSI七层体系结构：
  * 五层协议体系结构：网络接口层分为数据链路层 + 物理层

![](/images/network/network-protocol.png "协议对照")

![](/images/network/network-protocol-1.gif "各层协议对照")


> 一般网线传输的模拟信号需要通过猫(调制解调器)转换成数字信号，再经由路由器把信号发给PC端，如果PC端数量超过了路由器的连接上限就需要加装交换器。因此，家里有宽带就必须有猫，有多台电脑上网就必须要路由器，假如电脑很多，超过路由器的接口数就需要交换机扩展接口。

### 计算机网络
* 是网络通信技术与计算机技术相结合体，按照网络通信协议将计算机连接起来进行通信
* 连接介质可以是电缆、双绞线、光纤、微波、载波或通信卫星
* 分类：局域网（LAN）、城域网（MAN）、广域网（WAN）

### 互联网
* 全世界范围的计算机网络，信息社会的基石，使用TCP/IP协议
* 网络中的设备：交换机、路由器、服务器、PC、手机、物联网终端、车联网设备

## TCP与UDP协议
### TCP：传输控制协议 (Transmission Control Protocol)。
* 传输控制协议 (Transmission Control Protocol)，可以保证传输数据的安全，相对于UDP
* 面向连接的协议，先建立连接，再传输数据，传输的过程中还会保障传输数据的可靠性
* 建立连接三次握手，关闭连接四次挥手
* 应用场景：http请求都基于TCP进行数据传输，浏览网页，图片，下载文档
### UDP：用户数据报送协议(User Datagram Protocol)。
* 用户数据报送协议(User Datagram Protocol)，不保证数据传输的安全
* 面向无连接的协议，传输数据不需要建立连接，不管对方服务是否启动，发数据包就完了
* 传输数据速度快，安全性差，可能会丢失数据包
* 应用场景：视频会议，语音通话，视频直播…

### TCP-三次握手
三次握手：TCP协议在发送数据的准备阶段，客户端与服务器之间的三次交互，以保证连接的可靠
* 1次握手：Client发送带有 SYN 标志的数据包给Server
* 2次握手：Server发送带有 SYN/ACK 标志的数据包给Client
* 3次握手：Client发送带有 ACK 标志的数据包给Server【多此一举？】

![](/images/network/tcp-ws.png "TCP-三次握手")
#### 为什么要三次握手？
* 三次握手的目的是建立可靠的通信信道，双方确认自己与对方的发送与接收是正常的
* 1次握手： Client 什么都不能确认； Server 确认对方发送正常，自己接收正常
* 2次握手： Client 确认自己发送接收正常，对方发送接收正常；
* 3次握手： Server 确认自己发送正常，对方接收正常

### TCP-四次挥手
数据传输完毕，断开⼀个 TCP 连接则需要四次挥手：
* 1次挥手：Client发送⼀个 FIN给Server
* 2次挥手：Server收到这个FIN给Client发回一个ACK 
* 3次挥手：Server发送⼀个FIN给Client
* 4次挥手：Client收到这个FIN给Server发回一个ACK
![](/images/network/tcp-hs.png "TCP-四次挥手")
#### 为什么要四次挥手？
* 数据传输完毕，任何一方可以发送结束连接的通知，然后进入半关闭状态。另一方没有数据在发送的时候，则发
出结束连接的通知，对方确认后，完全关闭连接。
* 举个栗子：给女朋友打电话

## 输入URL地址到显示网页经历了哪些过程，会使用到哪些协议
![](/images/network/inputUrl.png "HTTP协议")

### 过程
1. 浏览器查找域名对应IP地址
  * DNS查找过程：浏览器缓存 ==> 主机HOST ==> 路由器缓存 ==> DNS缓存 ==> 根域名解析服务器
2. 浏览器想Web服务器发送一个HTTP请求，Cookies会随着请求发送给服务器
3. 服务器处理请求：获取参数和Cookies，处理请求生成一个html响应体
4. 服务器返回响应HTML
5. 浏览器渲染HTML
### 使用到的协议
* DNS协议：获取域名对应ip
* HTTP协议-应用层：使用HTTP协议访问网页，建立可靠连接与传输数据需要依靠TCP协议与IP协议
* TCP协议-传输层：与服务器建立TCP连接，并传输数据
* IP协议-网络层：建立TCP协议之后，发送数据在网络层依靠IP协议

## HTTP
![](/images/network/protocol_http_0.jpg "HTTP协议")

### HTTP1.0与HTTP1.1的区别是什么
#### 01-长连接
* HTTP/1.0默认使用的是短连接，HTTP1.1默认使用长连接，Connection:keep-alive
* HTTP/1.1长连接有非流水线方式和流水线（pipelining）方式
#### 02-错误状态码
* HTTP1.1新增24个错误状态响应码
#### 03-缓存
* HTTP1.0缓存判断标准单一
* HTTP1.1引入了更多的缓存控制策略
#### 04-断点续传
* HTTP1.0不支持断点续传，浪费带宽
* HTTP1.1加入断点续传支持，允许只请求资源的某个部分，充分利用带宽和连接

#### HTTP与HTTPS的区别是什么
#### 01-端口
* HTTP默认端口80
* HTTPS默认端口443
#### 02-协议
* HTTP的协议：http://
* HTTPS的协议： https://
#### 03-安全性与资源消耗
* HTTP安全性没有HTTPS高，资源消耗相比于HTTPS更低
* HTTPS是身披SSL（Secure Socket Layer）外壳的HTTP，HTTPS = HTTP + 加密 + 认证 + 完整性保护
* HTTP是明文传输数据，HTTPS是加密传输内容
* HTTPS传输内容加密使用对称加密算法，对称加密的密钥采用非对称加密

## URI与URL的区别是什么
URI（Uniform Resource Identifier）统一资源标志符：
* 资源抽象的定义，不管用什么方法表示，只要能定位一个资源就叫URI*。
* URL（Uniform Resource Locator）统一资源定位符，是一种具体的URI，在用地址定位
* URN（Uniform Resource Name）统一资源名称，也是一种具体的URI，在用名称定位

![](/images/network/URL.png "HTTP协议")

## DNS协议
![](/images/network/network-dns.png "DNS协议")

## 参考文章
1. [7层协议](https://www.pdai.tech/md/develop/protocol/dev-protocol-osi7.html "7层协议")
2. [HTTP协议](https://www.pdai.tech/md/develop/protocol/dev-protocol-http.html "HTTP协议")
3. [DNS协议](https://www.pdai.tech/md/develop/protocol/dev-protocol-dns.html "DNS协议")
4. [知识点串联：输入URL到页面加载过程详解](https://www.pdai.tech/md/develop/protocol/dev-protocol-url.html "知识点串联：输入URL到页面加载过程详解")