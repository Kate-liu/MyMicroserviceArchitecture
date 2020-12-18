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





































































