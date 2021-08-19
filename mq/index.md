# 消息队列


## 概述
消息队列（Message Queue，简称 MQ）是构建分布式互联网应用的基础设施，消息队列已经逐渐成为企业IT系统内部通信的核心手段。它具有低耦合、可靠投递、广播、流量控制、最终一致性等一系列功能，成为异步RPC的主要手段之一。

![](/images/mq/allmq.png "消息队列应用")

## 使用场景
### 业务解耦
解耦是消息队列要解决的最本质问题。所谓解耦，简单点讲就是一个事务，只关心核心的流程。而需要依赖其他系统但不那么重要的事情，有通知即可，无需等待结果。换句话说，基于消息的模型，关心的是“通知”，而非“处理”。 比如在美团旅游，我们有一个产品中心，产品中心上游对接的是主站、移动后台、旅游供应链等各个数据源；下游对接的是筛选系统、API系统等展示系统。当上游的数据发生变更的时候，如果不使用消息系统，势必要调用我们的接口来更新数据，就特别依赖产品中心接口的稳定性和处理能力。但其实，作为旅游的产品中心，也许只有对于旅游自建供应链，产品中心更新成功才是他们关心的事情。而对于团购等外部系统，产品中心更新成功也好、失败也罢，并不是他们的职责所在。他们只需要保证在信息变更的时候通知到我们就好了。 而我们的下游，可能有更新索引、刷新缓存等一系列需求。对于产品中心来说，这也不是我们的职责所在。说白了，如果他们定时来拉取数据，也能保证数据的更新，只是实时性没有那么强。但使用接口方式去更新他们的数据，显然对于产品中心来说太过于“重量级”了，只需要发布一个产品ID变更的通知，由下游系统来处理，可能更为合理。 再举一个例子，对于我们的订单系统，订单最终支付成功之后可能需要给用户发送短信积分什么的，但其实这已经不是我们系统的核心流程了。如果外部系统速度偏慢（比如短信网关速度不好），那么主流程的时间会加长很多，用户肯定不希望点击支付过好几分钟才看到结果。那么我们只需要通知短信系统“我们支付成功了”，不一定非要等待它处理完成。

### 异步处理
多应用对消息队列中同一消息进行处理，应用间并发处理消息，相比串行处理，减少处理时间；

### 限流削峰，错峰流控
试想上下游对于事情的处理能力是不同的。比如，Web前端每秒承受上千万的请求，并不是什么神奇的事情，只需要加多一点机器，再搭建一些LVS负载均衡设备和Nginx等即可。但数据库的处理能力却十分有限，即使使用SSD加分库分表，单机的处理能力仍然在万级。由于成本的考虑，我们不能奢求数据库的机器数量追上前端。 这种问题同样存在于系统和系统之间，如短信系统可能由于短板效应，速度卡在网关上（每秒几百次请求），跟前端的并发量不是一个数量级。但用户晚上个半分钟左右收到短信，一般是不会有太大问题的。如果没有消息队列，两个系统之间通过协商、滑动窗口等复杂的方案也不是说不能实现。但系统复杂性指数级增长，势必在上游或者下游做存储，并且要处理定时、拥塞等一系列问题。而且每当有处理能力有差距的时候，都需要单独开发一套逻辑来维护这套逻辑。所以，利用中间系统转储两个系统的通信内容，并在下游系统有能力处理这些消息的时候，再处理这些消息，是一套相对较通用的方式。

总而言之，消息队列不是万能的。对于需要强事务保证而且延迟敏感的，RPC是优于消息队列的。 对于一些无关痛痒，或者对于别人非常重要但是对于自己不是那么关心的事情，可以利用消息队列去做。 支持最终一致性的消息队列，能够用来处理延迟不那么敏感的“分布式事务”场景，而且相对于笨重的分布式事务，可能是更优的处理方式。 当上下游系统处理能力存在差距的时候，利用消息队列做一个通用的“漏斗”。在下游有能力处理的时候，再进行分发。 如果下游有很多系统关心你的系统发出的通知的时候，果断地使用消息队列吧。
广泛应用于秒杀或抢购活动中，避免流量过大导致应用系统挂掉的情况；

### 最终一致性
> 最终一致性不是消息队列的必备特性，但确实可以依靠消息队列来做最终一致性的事情。

## 投递模式
### 点对点模式（Point-to-Point， Queue）
Point-to-Point，点对点通信模型。PTP是基于队列(Queue)的，一个队列可以有多个生产者，和多个消费者。消息服务器按照收到消息的先后顺序，将消息放到队列中。队列中的每一条消息，只能由一个消费者进行消费，消费之后就会从队列中移除。

### 发布/订阅模式（publish/subscribe，topic）
* 每个消息可以有多个订阅者；
* 发布者和订阅者之间有时间上的依赖性。针对某个主题（Topic）的订阅者，它必须创建一个订阅者之后，才能消费发布者的消息。
* 为了消费消息，订阅者需要提前订阅该角色主题，并保持在线运行；

###  Partition模型
生产者发送消息到某个Topic中时，最终选择其中一个Partition进行发送。你可以将Parition模型中的分区，理解为PTP模型的队列，不同的是，PTP模型中的队列存储的是所有的消息，而每个Partition只会存储部分数据。
对于消息者，此时多了一个消费者组的概念，Paritition会在同一个消费者组下的消费者中进行分配，每个消费者只消费分配给自己的Paritition。上图演示了不同的消费者可能会分配到不同数量的Paritition。
Paritition模式巧妙的将PTP模型和Pub/Sub模型结合在了一起：

### Transfer模型
Paritition模型中的消费者组概念很有用，同一个Topic下的消息可以由多个不同业务方进行消费，只要使用不同的消费者组即可，不同消费者组消费到的位置单独记录，互不影响。 但是，Paritition模型还是限制了消费者数量不能多于分区数。

## 设计一个简单的消息队列
一般来讲，设计消息队列的整体思路是先build一个整体的数据流,例如producer发送给broker,broker发送给consumer,consumer回复消费确认，broker删除/备份消息等。 利用RPC将数据流串起来。然后考虑RPC的高可用性，尽量做到无状态，方便水平扩展。 之后考虑如何承载消息堆积，然后在合适的时机投递消息，而处理堆积的最佳方式，就是存储，存储的选型需要综合考虑性能/可靠性和开发维护成本等诸多因素。 为了实现广播功能，我们必须要维护消费关系，可以利用zk/config server等保存消费关系。 在完成了上述几个功能后，消息队列基本就实现了。然后我们可以考虑一些高级特性，如可靠投递，事务特性，性能优化等。 下面我们会以设计消息队列时重点考虑的模块为主线，穿插灌输一些消息队列的特性实现方法，来具体分析设计实现一个消息队列时的方方面面。

### RPC通信协议
刚才讲到，所谓消息队列，无外乎两次RPC加一次转储，当然需要消费端最终做消费确认的情况是三次RPC。既然是RPC，就必然牵扯出一系列话题，什么负载均衡啊、服务发现啊、通信协议啊、序列化协议啊，等等。在这一块，我的强烈建议是不要重复造轮子。利用公司现有的RPC框架：Thrift也好，Dubbo也好，或者是其他自定义的框架也好。因为消息队列的RPC，和普通的RPC没有本质区别。当然了，自主利用Memchached或者Redis协议重新写一套RPC框架并非不可（如MetaQ使用了自己封装的Gecko NIO框架，卡夫卡也用了类似的协议）。但实现成本和难度无疑倍增。排除对效率的极端要求，都可以使用现成的RPC框架。 简单来讲，服务端提供两个RPC服务，一个用来接收消息，一个用来确认消息收到。并且做到不管哪个server收到消息和确认消息，结果一致即可。当然这中间可能还涉及跨IDC的服务的问题。这里和RPC的原则是一致的，尽量优先选择本机房投递。你可能会问，如果producer和consumer本身就在两个机房了，怎么办？首先，broker必须保证感知的到所有consumer的存在。其次，producer尽量选择就近的机房就好了。

### 高可用
其实所有的高可用，是依赖于RPC和存储的高可用来做的。先来看RPC的高可用，美团的基于MTThrift的RPC框架，阿里的Dubbo等，其本身就具有服务自动发现，负载均衡等功能。而消息队列的高可用，只要保证broker接受消息和确认消息的接口是幂等的，并且consumer的几台机器处理消息是幂等的，这样就把消息队列的可用性，转交给RPC框架来处理了。 那么怎么保证幂等呢？最简单的方式莫过于共享存储。broker多机器共享一个DB或者一个分布式文件/kv系统，则处理消息自然是幂等的。就算有单点故障，其他节点可以立刻顶上。另外failover可以依赖定时任务的补偿，这是消息队列本身天然就可以支持的功能。存储系统本身的可用性我们不需要操太多心，放心大胆的交给DBA们吧！ 对于不共享存储的队列，如Kafka使用分区加主备模式，就略微麻烦一些。需要保证每一个分区内的高可用性，也就是每一个分区至少要有一个主备且需要做数据的同步，关于这块HA的细节，可以参考下篇pull模型消息系统设计。

### 服务端承载消息堆积的能力
消息到达服务端如果不经过任何处理就到接收者了，broker就失去了它的意义。为了满足我们错峰/流控/最终可达等一系列需求，把消息存储下来，然后选择时机投递就显得是顺理成章的了。 只是这个存储可以做成很多方式。比如存储在内存里，存储在分布式KV里，存储在磁盘里，存储在数据库里等等。但归结起来，主要有持久化和非持久化两种。 持久化的形式能更大程度地保证消息的可靠性（如断电等不可抗外力），并且理论上能承载更大限度的消息堆积（外存的空间远大于内存）。 但并不是每种消息都需要持久化存储。很多消息对于投递性能的要求大于可靠性的要求，且数量极大（如日志）。这时候，消息不落地直接暂存内存，尝试几次failover，最终投递出去也未尝不可。 市面上的消息队列普遍两种形式都支持。当然具体的场景还要具体结合公司的业务来看。

### 存储子系统的选择
我们来看看如果需要数据落地的情况下各种存储子系统的选择。理论上，从速度来看，文件系统>分布式KV（持久化）>分布式文件系统>数据库，而可靠性却截然相反。还是要从支持的业务场景出发作出最合理的选择，如果你们的消息队列是用来支持支付/交易等对可靠性要求非常高，但对性能和量的要求没有这么高，而且没有时间精力专门做文件存储系统的研究，DB是最好的选择。 但是DB受制于IOPS，如果要求单broker 5位数以上的QPS性能，基于文件的存储是比较好的解决方案。整体上可以采用数据文件+索引文件的方式处理，具体这块的设计比较复杂，可以参考下篇的存储子系统设计。 分布式KV（如MongoDB，HBase）等，或者持久化的Redis，由于其编程接口较友好，性能也比较可观，如果在可靠性要求不是那么高的场景，也不失为一个不错的选择。

### 消费关系解析
现在我们的消息队列初步具备了转储消息的能力。下面一个重要的事情就是解析发送接收关系，进行正确的消息投递了。 市面上的消息队列定义了一堆让人晕头转向的名词，如JMS 规范中的Topic/Queue，Kafka里面的Topic/Partition/ConsumerGroup，RabbitMQ里面的Exchange等等。抛开现象看本质，无外乎是单播与广播的区别。所谓单播，就是点到点；而广播，是一点对多点。当然，对于互联网的大部分应用来说，组间广播、组内单播是最常见的情形。 消息需要通知到多个业务集群，而一个业务集群内有很多台机器，只要一台机器消费这个消息就可以了。 当然这不是绝对的，很多时候组内的广播也是有适用场景的，如本地缓存的更新等等。另外，消费关系除了组内组间，可能会有多级树状关系。这种情况太过于复杂，一般不列入考虑范围。所以，一般比较通用的设计是支持组间广播，不同的组注册不同的订阅。组内的不同机器，如果注册一个相同的ID，则单播；如果注册不同的ID(如IP地址+端口)，则广播。 至于广播关系的维护，一般由于消息队列本身都是集群，所以都维护在公共存储上，如config server、zookeeper等。维护广播关系所要做的事情基本是一致的:
* 发送关系的维护。
* 发送关系变更时的通知。

## 队列高级特性设计

## 常用消息队列对比
![](/images/mq/mqvs.png "消息队列对比")

## 常见问题
### 消息去重
* 去重原则：保持幂等性，不管来多少条重复消息，最后处理的结果都一样，需要业务端来实现。
* 幂等性：就是用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用，数据库的结果都是唯一的，不可变的。
* 去重策略：保证每条消息都有唯一编号(比如唯一流水号)，且保证消息处理成功与去重表的日志同时出现。
* 幂等性具体方案：强一致性通过DB实现，弱一致性提高Reids实现。
    * 建立一个消息表，拿到这个消息做数据库的insert操作。给这个消息做一个唯一主键或者唯一约束，那么就算出现重复消费的情况，就会导致主键冲突，那么就不再处理这条消息。

### 消息丢失
* 生产者需要处理好Broker的响应，出错情况下利用重试、报警等手段。
* Broker需要控制响应的时机，单机情况下是消息刷盘后返回响应，集群多副本情况下，即发送至两个副本及以上的情况下再返回响应。
* 消费者需要在执行完真正的业务逻辑之后再返回响应给Broker。
* 但是要注意消息可靠性增强了，性能就下降了，等待消息刷盘、多副本同步后返回都会影响性能。因此还是看业务，例如日志的传输可能丢那么一两条关系不大，因此没必要等消息刷盘再响应。

### 消息最终一致性（分布式事务）
> 消息中间件可以作为用来实现分布式事务的一种手段，但其本身并不提供全局分布式事务的功能。

具体实现主要是用“记录”和“补偿”的方式。在做所有的不确定的事情之前，先把事情记录下来，然后去做不确定的事情，结果可能是：成功、失败或是不确定，“不确定”（例如超时等）可以等价为失败。成功就可以把记录的东西清理掉了，对于失败和不确定，可以依靠定时任务等方式把所有失败的事情重新搞一遍，直到成功为止。 回到刚才的例子，系统在A扣钱成功的情况下，把要给B“通知”这件事记录在库里（为了保证最高的可靠性可以把通知B系统加钱和扣钱成功这两件事维护在一个本地事务里），通知成功则删除这条记录，通知失败或不确定则依靠定时任务补偿性地通知我们，直到我们把状态更新成正确的为止。 整个这个模型依然可以基于RPC来做，但可以抽象成一个统一的模型，基于消息队列来做一个“企业总线”。 具体来说，本地事务维护业务变化和通知消息，一起落地（失败则一起回滚），然后RPC到达broker，在broker成功落地后，RPC返回成功，本地消息可以删除。否则本地消息一直靠定时任务轮询不断重发，这样就保证了消息可靠落地broker。 broker往consumer发送消息的过程类似，一直发送消息，直到consumer发送消费成功确认。 我们先不理会重复消息的问题，通过两次消息落地加补偿，下游是一定可以收到消息的。然后依赖状态机版本号等方式做判重，更新自己的业务，就实现了最终一致性。

## 参考文章
* [消息队列设计精要](https://tech.meituan.com/2016/07/01/mq-design.html "消息队列设计精要")
* [消息队列选型对比](https://www.infoq.cn/article/kafka-vs-rabbitmq/ "消息队列选型对比")