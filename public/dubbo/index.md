# Dubbo总结


> Apache Dubbo 是一款微服务开发框架，它提供了 RPC通信 与 微服务治理 两大关键能力。这意味着，使用 Dubbo 开发的微服务，将具备相互之间的远程发现与通信能力， 同时利用 Dubbo 提供的丰富服务治理能力，可以实现诸如服务发现、负载均衡、流量调度等服务治理诉求。同时 Dubbo 是高度可扩展的，用户几乎可以在任意功能点去定制自己的实现，以改变框架的默认行为来满足自己的业务需求。

服务是 Dubbo 中的核心概念，一个服务代表一组 RPC 方法的集合，服务是面向用户编程、服务发现机制等的基本单位。

Dubbo 开发的基本流程是：用户定义 RPC 服务，通过约定的配置 方式将 RPC 声明为 Dubbo 服务，然后就可以基于服务 API 进行编程了。对服务提供者来说是提供 RPC 服务的具体实现，而对服务消费者来说则是使用特定数据发起服务调用。

## 服务发现
> 服务发现，即消费端自动发现服务地址列表的能力，是微服务框架需要具备的关键能力，借助于自动化的服务发现，微服务之间可以在无需感知对端部署位置与 IP 地址的情况下实现通信。

实现服务发现的方式有很多种，Dubbo 提供的是一种 Client-Based 的服务发现机制，通常还需要部署额外的第三方注册中心组件来协调服务发现过程，如常用的 Nacos、Consul、Zookeeper 等，Dubbo 自身也提供了对多种注册中心组件的对接，用户可以灵活选择。

![](/images/dubbo/architecture.png "服务发现")

服务发现的一个核心组件是注册中心，Provider 注册地址到注册中心，Consumer 从注册中心读取和订阅 Provider 地址列表。 因此，要启用服务发现，需要为 Dubbo 增加注册中心配置：

以 dubbo-spring-boot-starter 使用方式为例，增加 registry 配置
```yml
# application.properties
dubbo
 registry
  address: zookeeper://127.0.0.1:2181
```

## 服务流量管理
流量管理的本质是将请求根据制定好的路由规则分发到应用服务上，如下图所示：
![](/images/dubbo/what-is-traffic-control.png "流量管理")

Dubbo提供了支持mesh方式的流量管理策略，可以很容易实现 A/B测试、金丝雀发布、蓝绿发布等能力。

Dubbo将整个流量管理分成VirtualService和DestinationRule两部分。当Consumer接收到一个请求时，会根据VirtualService中定义的DubboRoute和DubboRouteDetail匹配到对应的DubboDestination中的subnet，最后根据DestinationRule中配置的subnet信息中的labels找到对应需要具体路由的Provider集群。其中：
* VirtualService主要处理入站流量分流的规则，支持服务级别和方法级别的分流。
* DubboRoute主要解决服务级别的分流问题。同时，还提供的重试机制、超时、故障注入、镜像流量等能力。
* DubboRouteDetail主要解决某个服务中方法级别的分流问题。支持方法名、方法参数、参数个数、参数类型、header等各种维度的分流能力。同时也支持方法级的重试机制、超时、故障注入、镜像流量等能力。
* DubboDestination用来描述路由流量的目标地址，支持host、port、subnet等方式。
* DestinationRule主要处理目标地址规则，可以通过hosts、subnet等方式关联到Provider集群。同时可以通过trafficPolicy来实现负载均衡。

这种设计理念很好的解决流量分流和目标地址之间的耦合问题。不仅将配置规则进行了简化有效避免配置冗余的问题，还支持VirtualService和DestinationRule的任意组合，可以非常灵活的支持各种业务使用场景。

## 参考文章
* [Dubbo 文档](https://dubbo.apache.org/zh/docs/)
* [一文带你搞懂RPC核心原理](https://mp.weixin.qq.com/s/3-i9aYyEb4z58fXxDwDpuA)
* [Dubbo技术详细介绍 ](https://mp.weixin.qq.com/s?__biz=MzI1NDQ3MjQxNA==&mid=2247483791&idx=1&sn=49345f1a022734e81e9257f2b8d38a52)
* [Dubbo技术详细介绍 ](http://www.buildupchao.cn/arch/2019/02/01/design-a-distributed-RPC-structure.html)
