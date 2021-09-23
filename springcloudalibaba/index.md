# Spring Cloud Alibaba

  

![](/images/spring/springcloudalibaba/SpringCloudAlibabaAll.png "Spring Cloud Alibaba")

## 组件
* Spring Cloud - Gateway 网关
* Spring Cloud - Ribbon 实现负载均衡
* Spring Cloud - Feign 实现远程调用
* Spring Cloud - Sleuth 实现调用链监控
* Spring Cloud Alibaba - Nacos 实现注册中心/配置中心 
* Spring Cloud Alibaba - Sentinel  实现服务容错(限流，降级)
* Spring Cloud Alibaba - Seata 实现分布式事务

![](/images/spring/springcloudalibaba/SpringCloudAlibaba.png "Spring Cloud Alibaba 组件")

## Nacos
> Nacos 是一个 Alibaba 开源的、易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

Nacos 这个名字怎么读呢？它的音标为 /nɑ:kəʊs/。这个名字不是一个标准的单词，而是以下单词的首字母缩写：Name and Config Service。

### Nacos Discovery
> 使用 Spring Cloud Alibaba Nacos Discovery，可基于 Spring Cloud 的编程模型快速接入 Nacos 服务注册功能。

### 配置中心 Nacos Config
> 使用 Spring Cloud Alibaba Nacos Config，可基于 Spring Cloud 的编程模型快速接入 Nacos 配置管理功能。

## Sentinel
> Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:
* 丰富的应用场景： Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、实时熔断下游不可用应用等。
* 完备的实时监控： Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
* 广泛的开源生态： Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
* 完善的 SPI 扩展点： Sentinel 提供简单易用、完善的 SPI 扩展点。您可以通过实现扩展点，快速的定制逻辑。例如定制规则管理、适配数据源等。

## 学习资料
* [Spring Cloud Alibaba 参考文档](https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/zh-cn/index.html)
* [Nacos](https://nacos.io/zh-cn/docs/what-is-nacos.html)
* [Sentinel](https://sentinelguard.io/zh-cn/docs/quick-start.html)
* [SkyWalking 分布式追踪](https://skywalking.apache.org/zh/2020-04-19-skywalking-quick-start/)
