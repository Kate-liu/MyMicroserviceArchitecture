# Microservice Distributed System

## 如何设计一个分布式技术服务-系统设计面试案例

### 需求设计和简化架构

#### 面试题

如何设计一个视频观看量，关注量，应用请求量的技术系统。

![1608216549440](MicroserviceDistributedSystem.assets/1608216549440.png)



#### 需求沟通

考察点：主动性和沟通能力。

由于不同技术背景的人，熟悉的技术不一样，所以需要进一步明确需求。

![1608216619896](MicroserviceDistributedSystem.assets/1608216619896.png)



#### 需求澄清

- 场景用例
- 量级规模
- 性能
- 成本

![1608216664230](MicroserviceDistributedSystem.assets/1608216664230.png)



#### 功能需求-API

- 处理需求，对视频观看计数
- 查询需求：按照时间段返回视频观看数量

![1608216726766](MicroserviceDistributedSystem.assets/1608216726766.png)





#### 非功能需求

- 规模
- 性能
- 高可用
- 水平按需扩展
- 低成本

![1608216770562](MicroserviceDistributedSystem.assets/1608216770562.png)



#### 从简化架构开始

- 抽象出的服务：计数服务，查询服务
- 操作的是数据：数据库

![1608216826944](MicroserviceDistributedSystem.assets/1608216826944.png)



### 存储设计

#### 存什么？

- 单个事件的数据
- 聚合数据

![1608255189579](MicroserviceDistributedSystem.assets/1608255189579.png)



#### 数据库选型

- 可扩展
- 高性能
- 高可用
- 一致性
- 成本
- 学习门槛

![1608255225209](MicroserviceDistributedSystem.assets/1608255225209.png)



#### SQL数据库+客户端嵌入代理

- ShardingSphere
- 将计数服务，路由到MySQL主节点上，写入数据
- 将查询服务，路由到MySQL从结点上，读取数据

![1608255260243](MicroserviceDistributedSystem.assets/1608255260243.png)



#### SQL数据库+独立部署代理层

- ShardingSphere
- 记录数据库存储配置，有一个配置注册中心
- 分别进行计数服务和查询服务的路由

![1608255291715](MicroserviceDistributedSystem.assets/1608255291715.png)



#### NoSQL数据库（Cassandra）

- 随机选择一个Node进行判断，写入的Node
- 仲裁写，多数写好，就表示已经写好了
- 一致性读，多个数据中心复制备份

![1608255314164](MicroserviceDistributedSystem.assets/1608255314164.png)



#### 表设计

- SQL数据库，表之间的关联，join
- NOSQL数据库，数据冗余，

![1608255372136](MicroserviceDistributedSystem.assets/1608255372136.png)



### 计数服务设计

#### 计数服务如何实现

- 可扩展性
- 高性能
- 高可用

![1608255400853](MicroserviceDistributedSystem.assets/1608255400853.png)



#### 数据聚合（aggregation）基础

- 一般采用预聚合
- 请求pull，拉模式，引入消息队列

![1608255479430](MicroserviceDistributedSystem.assets/1608255479430.png)



#### 消息队列基础

- Kafka
- 类似于数组，偏移量，消费指针，检查点
- 分区，对同一个主题，开多个分区，进行消息分摊

![1608255505044](MicroserviceDistributedSystem.assets/1608255505044.png)



#### 计数消费者（详细设计）

- 数据聚合运算，使用内存中的concurrentHashmap，进行并发的运算
- 引入Internal的消息队列，作为数据库的缓冲
- DB writer 暂时无法写入数据到 DB 的时候，需要将数据写入 死信队列，保证数据不丢失
- 引入 Enrich Data Cache，将数据进行缓存，字段都写上值之后，在写入到数据库

![1608255531122](MicroserviceDistributedSystem.assets/1608255531122.png)



#### 数据接收路径（Data Ingestion Path）

- API Gateway
- Counting Service，服务代理
- Kafka

![1608255581090](MicroserviceDistributedSystem.assets/1608255581090.png)



#### 数据接收路径上的面试题

- API Gateway，软件Nginx，硬件F5，NDS，注册中心
- 容错限流，Histrix，TopK实时防爬虫
- 消息队列的格式，json，二进制

![1608255605252](MicroserviceDistributedSystem.assets/1608255605252.png)



### 查询服务设计

#### 数据获取路径（Data Retrieval Path）

- 查询服务 Query Services
- 老数据归档（对象存储），2/8原则，近期数据缓存
- 热数据，冷数据

![1608255671411](MicroserviceDistributedSystem.assets/1608255671411.png)



### 技术栈选型

#### 总体流程

- Counting service，计数服务
- Query service，查询服务

![1608255749951](MicroserviceDistributedSystem.assets/1608255749951.png)



#### 技术栈选型

- API Gateway：f5，zuul
- Spring Boot
- Kafka
- redis
- cassandra，hadoop

![1608255841307](MicroserviceDistributedSystem.assets/1608255841307.png)



### 进一步考量和总结

#### 更多面试问题

- 如何定位系统瓶颈？
- 如何监控系统健康状况？（日志，metric，调用链）
- 如何确保线上系统运行结果正确？（两套系统，实时流处理系统+线下批处理系统）
- 如何解决热分区问题？（对视频按照时间分摊）
- 如何监控慢消费？解决？





#### 总结

- 功能需求
- 非功能需求
- 总体设计
- 详细设计
- 评估

![1608255896004](MicroserviceDistributedSystem.assets/1608255896004.png)





#### 扩展

- 监控系统
- 欺诈检测系统
- 限流系统
- 推荐系统
- 今日热点



### 参考

- System Design Interview – Step By Step Guide：https://www.youtube.com/watch?v=bUHFg8CZFws
- The System Design Primer：https://github.com/donnemartin/system-design-primer
- Consistent Hash Implementation in Java：https://github.com/Jaskey/ConsistentHash





## 如何设计一个简化版Kafka消息队列-拍拍贷PMQ设计演进案例

### 消息队列PMQ 2.0的项目背景

#### 挑战1~PMQ 1.0（2016.8）

- 一套Kafka 系统
- 一套 redis 系统

![1608302197273](MicroserviceDistributedSystem.assets/1608302197273.png)



#### 挑战2

- 业务驱动
- 微服务与事件驱动架构

![1608302267215](MicroserviceDistributedSystem.assets/1608302267215.png)



#### 为啥不要Meta Q /Rocket MQ?

- 阿里 Rocket MQ
- 无 C# 客户端，通讯协议复杂，定制 C# 客户端非常困难
- 复杂
  - 依赖 ZK/ NameServer
  - 支持事务消息
- 代码质量一般



#### 为什么不要 Kafka？

- 重且复杂
  - 消息 HA 多份拷贝存储，Leader/Follower 协议
  - 依赖 ZK
  - 代码复杂，Scala 写，普通研发无法深入理解，定制困难
- 不支持企业治理功能，如Topic和业务团队关系管理等
- 不支持高级消息队列特性，比如查消息
- 基于文件存储，适合日志场景，业务场景需要深度定制
- 定制 Kafka 例子
  - https://github.com/allegro/hermes
  - https://github.com/zalando/nakadi



#### 为啥要造轮子？

- 业务消息数据太关键
- 个人原因，需要落地
- 个人背景
  - 携程日志采集平台
  - 个人开源项目 bugqueue/luxun
  - https://github.com/bulldog2011
- 携程 Hermes/ qmq的启发
  - https://github.com/qunarcorp/qmq
  - https://github.com/ctripcorp/hermes





#### 设计目标

![1608302592162](MicroserviceDistributedSystem.assets/1608302592162.png)





#### 设计限制

- 消息顺序性
  - 能保证消息顺序插入，保证相同分区的消息是顺序的（排除网络延迟），但是多个分区之间可能是乱序的
  - 消息并行消费或者多个分区并行消费，消息消费顺序可能是乱序的
- 消费语义
  - At Least Once 至少一次交付
  - 消费者挂或者重启，没有及时提交消费偏移，重启后可能接收到少量重复消息，消费者端业务方要做幂等处理







### PMQ 2.0的设计

#### 理解队列 Queue

- 入队，出队
- 头指针，尾指针
- https://github.com/bulldog2011/bigqueue

![1608352437701](MicroserviceDistributedSystem.assets/1608352437701.png)



#### Queue 和 Topic（Fanout）语义

-  不同的头指针，Fanout

![1608352526158](MicroserviceDistributedSystem.assets/1608352526158.png)



#### 核心概念模型

- 对 Queue 进行分区（Partution）
- 生产者与消费者关系

![1608352639474](MicroserviceDistributedSystem.assets/1608352639474.png)





#### 理解消费者组

- Consumer Group
- 消费组之间互不影响，消费进度各不相同
- 限定，每一个消费者组中的消费者，不能消费同一个Topic的同一个Partition

![1608352740442](MicroserviceDistributedSystem.assets/1608352740442.png)





#### 存储设计

- 数据结点 DataNode（数据库），创建 Patation（数据库表）
- 绑定到Topic
- 最多 99个 Patation

![1608352817851](MicroserviceDistributedSystem.assets/1608352817851.png)



#### 分区队列实现

- 使用数据库实现
- 数据库表字段，自增主键，索引

![1608352867488](MicroserviceDistributedSystem.assets/1608352867488.png)



#### 元数据模型

- 数据之间的关系

![1608352883088](MicroserviceDistributedSystem.assets/1608352883088.png)



#### PMQ 2.0 总体架构

- Broker 节点，是无状态的，按需扩展

![1608352948835](MicroserviceDistributedSystem.assets/1608352948835.png)



#### PMQ 2.0 服务发现

- 分布式系统，具体服务发现功能

![1608353040481](MicroserviceDistributedSystem.assets/1608353040481.png)





#### Push vs Pull 模式

- 推模式 与 拉模式
- 邮局的邮件 送与取
- 消费者拉模式，较好，最佳实践

![1608359886431](MicroserviceDistributedSystem.assets/1608359886431.png)



#### Kafka 总体架构

- 拉模式
- 使用 zookper

![1608359918007](MicroserviceDistributedSystem.assets/1608359918007.png)



#### Produces 和 Partition之间的负载均衡

- 保证顺序消费
- Hash 机制可保证相同 key 的消息发往同一个分区，消费者保证顺序消费同一个分区

![1608360037613](MicroserviceDistributedSystem.assets/1608360037613.png)



#### 同一组的 Consumer和Partition之间的负载均衡

- PMQ 2.0 暂时不支持动态重平衡，采用简单分区竞争策略，一个消费者一个分区
- 保证消费者不小于分区的数量，多部署一个消费者

![1608360088266](MicroserviceDistributedSystem.assets/1608360088266.png)



#### 什么是动态重平衡

- 增加消费者，实现分区的动态重新分配

![1608360162463](MicroserviceDistributedSystem.assets/1608360162463.png)



#### 消费者 Internal

- 拉消息 API
- 提交偏移 API
- 心跳检测 API
- 会有消息的重复消费

![1608360209789](MicroserviceDistributedSystem.assets/1608360209789.png)



#### 同步和异步 Produce

- 异步性能 >> 同步
- 异步模式，可以实现批量操作，存在批量等待延时

![1608360255449](MicroserviceDistributedSystem.assets/1608360255449.png)



#### HA 保证

- Broker 无状态
- MySQL 消息数据库有DMA通过主从保证HA
- Broker定期同步元数据库中的元数据，元数据库挂，仅影响元数据管理功能,现有生产和消费无影响。
- 管理界面挂，仅影响元数据管理功能，现有生产和消费无影响
- 生产者挂，通过Metrics监控告警提醒
- 消费者挂，或者慢消费(lag监控)，通过Metrics监控告警提醒



#### Lag 堆积监控

- 消费者速度跟不上生产者速度，存量多
- Lag 监控告警
- https://github.com/linkedin/Burrow

![1608361092014](MicroserviceDistributedSystem.assets/1608361092014.png)



#### 性能和扩展性

- 消费者拉模式吞吐第一
  - 消费支持批量拉(batch pull)，只要消费足够快，理论无延迟(消息越多越快)
- 同步生产理论无延迟，异步生产秒级延迟
- 数据库插入不锁表，异步插入性能更好，除自增顺序号无其它索引
- 生产者快,或者消费慢，扩容增加分区队列分摊负载
- 同步生产慢，可以开启异步生产
- DB节点压力大，增加更多DB节点分摊负载
  - 支持分区迁移(设置分区为只读)



#### 性能测试分析

- 同步生产，普通单机测试可达1k/秒吞吐(消息1k)
- 异步生产，普通单机测试可达10k/秒吞吐(消息1k, batch=10)
- 批量消费，普通单机测试可达10k/秒吞吐(消息1k, batch=10)
- 异步生产性能>>优于同步
  - 实时事务型应用，建议同步，但是吞吐量降低
  - 吞吐型日志型应用，建议异步，但是有秒级延迟



#### 隔离性

- 分区通过表隔离，一个分区对应一个表，不同主题/分区互不干扰
- 消费端拉消息, Broker无状态，消费端天然隔离
- 消费者组维护各自的消费偏移,互不干扰



#### PMQ 2.0  vs Kafka

- PMQ 2.0  是简化版的 Kafka

|                | PMQ 2.0                                       | Kafka                        |
| -------------- | --------------------------------------------- | ---------------------------- |
| 队列持久化     | MySQL                                         | 文件                         |
| 通讯层         | Thrift                                        | 定制NIO协议                  |
| 元数据分区管理 | MySQL+静态分配                                | ZK动态管理                   |
| 消费者负载均衡 | 通过ip+进程号竞争分配 （1 consumer <> 1 queue | 动态重平衡                   |
| 消费状态存储   | 客户端+MySQL                                  | 客户端+ZK或Broker            |
| 消息HA         | 依赖MySQL HA                                  | 在不同Broker上存多份消息拷贝 |







### PMQ 3.0 的演进

#### PMQ 3.0 规划（2018）





























