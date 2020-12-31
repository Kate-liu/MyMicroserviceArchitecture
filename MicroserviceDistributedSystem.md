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

- 消费者动态重平衡(P1)
- 调整消费偏移无需重启消费者(P1)
- 运维治理→研发自助(P1)
-  .Net转Java(P1)
- 完善失败消息处理(P1)
- 延迟消息(P2)
- 生产者异步持久化(P2)
- 开源(P2)
- 灵活消息查询ELK(P3)
- 消息轨迹可视化(P3)
- 事务消息?(P3)
- https://github.com/ppdaicorp/pmq
- https://github.com/ppdaicorp/pmq/wiki



#### 当前生产细节（PMQ 3.0/2020.5）

- 当前15(5主+10从)个DB物理机(4Oc)，20台Broker虚拟机。
- 一个DB节点10个库，一个库99张表，一个DB支持990个分区
- 消息保持7天后删除老消息(定时器)
- 单表100万消息OK，7天可支持700~1000万消息
- 分区个数估算，先估算topic每日消息量，除以100万，例如预估日700万消息，则预分配7个分区队列。测试环境一律2个分区。
- 日消息发送3亿，消费 6亿





### Kafka的动态重平衡是如何工作的？

#### Kafka Rebalance Protocol

- 分布式算法：动态组成员的资源分配问题
- Rebalance/Rebalancing: the procedure that is followed by a number of distributed processes that use Kafka clients and/or the Kafka coordinator to form a common group and distributea set of resources among the member of the group (source :lncremental Cooperative Rebalancing: Support and Policies)



#### 分区-消费者场景

- 对不同的分区，根据消费者的个数，动态分配

![1608362893189](MicroserviceDistributedSystem.assets/1608362893189.png)



#### Kafka 重平衡协议和组件

- 自定义实现重平衡协议

![1608362965273](MicroserviceDistributedSystem.assets/1608362965273.png)



#### Protocol：JoinGroup Request

- 选出协调者 Coordinator，保证确定消费者的状态，组织消费者

![1608363083449](MicroserviceDistributedSystem.assets/1608363083449.png)



#### Protocol：JoinGroup Response

- 协调者 Coordinator 等一会，类似于屏障（barrier），
- 当消费者不变的时候，再从消费者中选出一个 Leader

![1608363154029](MicroserviceDistributedSystem.assets/1608363154029.png)



#### Protocol：SyncGroup Request

- 进行不同分区的请求

![1608363292631](MicroserviceDistributedSystem.assets/1608363292631.png)



#### Protocol：SyncGroup Response

- 进行不同分区的响应

![1608363322377](MicroserviceDistributedSystem.assets/1608363322377.png)



#### Protocol：Heartbeat

- 心跳检测，保证服务正常

![1608363392910](MicroserviceDistributedSystem.assets/1608363392910.png)





#### Protocol：LeaveGroup

- stop the world 效应
- 当有消费者离开消费组的时候，全部消费者停止消费，重新进行分配，重新进行分配周期

![1608363475509](MicroserviceDistributedSystem.assets/1608363475509.png)



#### Rebalance

- 当经过了充分分配的周期之后，进入了新的平衡状态

![1608363575569](MicroserviceDistributedSystem.assets/1608363575569.png)



#### JoinGroup Again

- 当某个消费者，又连接成功到 coordinator的时候
- 重新进行分区与消费者分配，再一次 Stop the world

![1608363689530](MicroserviceDistributedSystem.assets/1608363689530.png)



#### Timeout

- 根据两个参数判断，消费者是否死掉

![1608363742577](MicroserviceDistributedSystem.assets/1608363742577.png)



#### Kafka 重平衡优化

- Static Membership
- Incremental Cooperative Rebalabcing
- Apache Kafka Rebalance Protocol, or the magic behind your streams applications：https://medium.com/streamthoughts/apache-kafka-rebalance-protocol-or-the-magic-behind-your-streams-applications-e94baf68e4f2



#### 动态重平衡 in PMQ 3.0

- 简化版动态重平衡

![1608364516550](MicroserviceDistributedSystem.assets/1608364516550.png)



### 消息队列设计和治理最佳实践

#### 推Push vs 拉Pull

- 推，Broker 有状态，客户端简单
- 拉，Broker 无状态，将复杂性放在消费端

![1608359886431](MicroserviceDistributedSystem.assets/1608359886431.png)



#### 泳道和舱壁隔离

- 隔离，功能与业务
- Hystrics

![1608365171364](MicroserviceDistributedSystem.assets/1608365171364.png)



#### 延迟 vs 吞吐

![1608365203911](MicroserviceDistributedSystem.assets/1608365203911.png)



#### 动态重平衡协议

![1608363154029](MicroserviceDistributedSystem.assets/1608363154029.png)



#### 核心基础设施要自研 or 定制

- PMQ 2.0  是简化版的 Kafka

|                | PMQ 2.0                                       | Kafka                        |
| -------------- | --------------------------------------------- | ---------------------------- |
| 队列持久化     | MySQL                                         | 文件                         |
| 通讯层         | Thrift                                        | 定制NIO协议                  |
| 元数据分区管理 | MySQL+静态分配                                | ZK动态管理                   |
| 消费者负载均衡 | 通过ip+进程号竞争分配 （1 consumer <> 1 queue | 动态重平衡                   |
| 消费状态存储   | 客户端+MySQL                                  | 客户端+ZK或Broker            |
| 消息HA         | 依赖MySQL HA                                  | 在不同Broker上存多份消息拷贝 |



#### MQ 治理最佳实践

- 研发自助治理
  - 主题/分区申请，扩容，监控
- 堆积(lag)监控告警
- 动态偏移调整
- 失败消息处理
  - 死信(dead letter)队列
- 线上测试+监控
  - https://github.com/linkedin/kafka-monitor



## 如何解决微服务的数据一致性和事务问题

### 微服务的四大技术难题是什么？

#### 微服务四大技术难题

> 最难的部分和数据（状态）有关

- 数据一致性分发
- 数据聚合Join
- 分布式事务
- 单体系统解耦拆分

![1608365934145](MicroserviceDistributedSystem.assets/1608365934145.png)



### 如何解决微服务的数据一致性分发问题？

#### 为啥要分发数据？场景？

- 更新缓存
- 同步到其他服务
- 同步到搜索引擎
- 同步到数据仓库
- 数据复制（replication）
- 支持数据库拆分迁移
- 实现CQRS/去数据库Join
- 实现分布式事务 
- 流计算
- 大数据BI/AI
- 审计日志，历史归档

![1608373971899](MicroserviceDistributedSystem.assets/1608373971899.png)



#### 双写？

- 双写的时候，如何保证事务性？

![1608374022625](MicroserviceDistributedSystem.assets/1608374022625.png)





#### 模式一：事务性发件箱（Transactional Outbox）

- 保证两个动作的事务性
- 需要在 MQ 实现 幂等性

![1608374065703](MicroserviceDistributedSystem.assets/1608374065703.png)



#### Transactional Outbox 参考实现~Killbill Common Queue

- 基于集中式数据库实现
- https://github.com/killbill/killbill-commons/tree/master/queue

![1608374159420](MicroserviceDistributedSystem.assets/1608374159420.png)



#### Reaper机制

- 收割机线程，Reaper
- 查看无人处理的任务，标记为自己的任务，保证高可用
- https://github.com/killbill/killbill-commons/tree/master/queue

![1608374210482](MicroserviceDistributedSystem.assets/1608374210482.png)



#### Killbill PersistentBus表结构

- https://github.com/killbill/killbill-commons/blob/master/queue/src/main/resources/org/killbill/queue/ddl.sql

![1608374370013](MicroserviceDistributedSystem.assets/1608374370013.png)



#### Killbill PersistentBus处理状态迁移

- 处理状态的状态图
- 有失误状态重试最大次数

![1608374397396](MicroserviceDistributedSystem.assets/1608374397396.png)



#### 模式二：变更数据捕获（Change Data Capture，CDC）

- 使用变更的日志，进行捕获
- 实现两个事件的事务

![1608374456585](MicroserviceDistributedSystem.assets/1608374456585.png)



#### CDC开源项目（企业级）

- 推荐生产使用，阿里 Canal
- https://github.com/alibaba/canal
- Redhat Debezium
- https://github.com/debezium/debezium
- Zzendesk Maxwell
- https://github.com/zendesk/maxwell
- Airbnb SpinalTap
- https://github.com/airbnb/SpinalTap

![1608374544196](MicroserviceDistributedSystem.assets/1608374544196.png)





#### 学习参考~Eventuate-Tram

- 微服务架构设计模式
- http://www.chrisrichardson.net/
- https://eventuate.io/

![1608374921366](MicroserviceDistributedSystem.assets/1608374921366.png)



#### Transactional Outbox vs CDC

|                   | Transaction Outbox     | CDC                              |
| ----------------- | ---------------------- | -------------------------------- |
| 复杂性            | 相对简单               | 复杂（高可用/监控）              |
| Pulling延迟和开销 | 近实时，有一定性能开销 | 较实时，性能开销小               |
| 应用侵入性        | 有                     | 无                               |
| 适合场合          | 早期/中小规模          | 中大规模，有独立框架团队治理维护 |



#### Single Source of Truth

- 也称Single System of Record
  - 某一个服务是某些数据的唯一主人
  - 该服务是数据的权威记录系统(canonical system of record)
  - 其它的数据拷贝都是只读，非权威的缓存(read-only, non-authoritative cache)

![1608375164408](MicroserviceDistributedSystem.assets/1608375164408.png)



### 如何解决微服务的数据聚合Join问题？

#### 单库Join的问题

- join 会引入 笛卡尔积等

![1608376126838](MicroserviceDistributedSystem.assets/1608376126838.png)



#### 分布式聚合Join的问题

- 聚合层（Aggregator），BFF（Backend for Frontend）层
- N+1问题，一个数据库查一次，一个数据库查n次
- 数据量问题
- 性能开销问题

![1608376219808](MicroserviceDistributedSystem.assets/1608376219808.png)



#### Denormalize + Materialize the View

- 数据分发技术
- 反正规化 + 物化视图
- 预聚合技术

![1608376256075](MicroserviceDistributedSystem.assets/1608376256075.png)



#### CQRS（Command Query ）模式

- 服务层的读写模式
- 命令查询的责任分离 Command Query Responsibility Segregation (简称CQRS)

![1608376361264](MicroserviceDistributedSystem.assets/1608376361264.png)



#### CQRS 和最终一致性

- UI 更新问题
- At Least Once 语义
  - 客户端幂等
- 最终一致性，引入时间差问题

![1608376420058](MicroserviceDistributedSystem.assets/1608376420058.png)



#### CQRS 和 UI 更新策略

- 乐观更新 Optimistic update
- 拉模式 Pull
- 发布订阅模式 Publish-subscribu

![1608376490100](MicroserviceDistributedSystem.assets/1608376490100.png)



#### 网站架构 2005 vs 2016

- 2016 年架构只是在 2005年的基础上变得更加复杂而已，种类更多，基本流程还是没有变化的

![1608376567328](MicroserviceDistributedSystem.assets/1608376567328.png)



### 如何解决微服务的分布式事务问题？

#### 单机DB事务 -> 分布式事务

- 订单表与库存表，一个事务保证操作没错

![1608387865746](MicroserviceDistributedSystem.assets/1608387865746.png)



#### ACID事务保证

- 原子性 Atomicity
- 一致性 Consistency
- 隔离性 Isolation
- 持久性 Durability

![1608387921391](MicroserviceDistributedSystem.assets/1608387921391.png)



#### 事务隔离级别

- 读未提交 Read Uncommited
- 读已提交 Read Committed
- 可重复读 Repeatable Reads（MySQL默认隔离级别）
- 串行化  Serializable
- 分布式事务从0到1-认识分布式事务：https://www.codingapi.com/blog/2020/01/01/txlcn001/

![1608387991801](MicroserviceDistributedSystem.assets/1608387991801.png)



#### 2PC/XA（两阶段提交）

- 两阶段事务
- 第一阶段，准备阶段
- 第二阶段，提交事务，或者回滚事务
- XA，分布式两阶段提交，一个二阶段提交协议的规范
- 阿里 Seate 是一种优化版 2PC

![1608388332042](MicroserviceDistributedSystem.assets/1608388332042.png)



#### 2PC 样例

- 基于 Atomikos 实现股票交易分布式事务
- 分布式事务样例：Atomikos + Spring + MySQL + ActiveMQ + DerbySpring
- 事务传播行为：https://github.com/Apress/practical-microservices-architectural-patterns/tree/master/Christudas_Ch13_Source/ch13/ch13-01/XA-TX-Distributed

![1608388401649](MicroserviceDistributedSystem.assets/1608388401649.png)



#### TCC

- 一种2PC变体， 约等于 应用层/服务层 2PC
- 另一种分布式事务解决方案
- TCC 简化流程，每一个服务需要实现三个接口，Try接口，confirm接口，Cancel接口

![1608388572524](MicroserviceDistributedSystem.assets/1608388572524.png)



#### CAP 原理

- 系统分区之后，无法同时满足A与C
- CAP：一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance）

![1608389678704](MicroserviceDistributedSystem.assets/1608389678704.png)



#### 成年人不用分布式事务（2PC）

- Life Beyond Distributed Transactions -  An apostate's opinion
- 超越分布式事务 - 一个反叛者的观点
- https://queue.acm.org/detail.cfm?id=3025012

![1608389813674](MicroserviceDistributedSystem.assets/1608389813674.png)



#### 微服务时代的事务处理原则

> Saga  模式，分布式事务的解决方案
>
> - 将全局事务建模成一组本地ACID事务
> - 引入事务补偿机制处理失败场景

- 假定网络或者服务不可靠
- 将全局事务建模成一组本地ACID事务
- 引入事务补偿机制处理失败场景
- 事务始终处在一种明确的状态(不管成功还是失败)
- 最终一致
- 考虑隔离性
- 考虑幂等性
- 异步响应式，尽量避免直接同步调用



#### 购物场景状态机（简化）

- 简化事务状态机
- 状态机有明确的状态迁移方案

![1608389905489](MicroserviceDistributedSystem.assets/1608389905489.png)



#### 协同式（Choreography）Saga

- 发消息模式

![1608389924425](MicroserviceDistributedSystem.assets/1608389924425.png)



#### 编排式（Orchestration）Saga

- 有一个编排者，让后续系统，创建事务，做什么事
- 集中的方式

![1608389935174](MicroserviceDistributedSystem.assets/1608389935174.png)



#### 补偿样例1

![1608389989501](MicroserviceDistributedSystem.assets/1608389989501.png)



#### 补偿样例2

![1608390003586](MicroserviceDistributedSystem.assets/1608390003586.png)



#### 协同式（Choreography）vs 编排式（Orchestration）

- 协同式（Choreography），适合小系统
- 编排式（Orchestration），适合大系统，扩展方便
- 芭蕾舞 vs 交响乐

![1608390038636](MicroserviceDistributedSystem.assets/1608390038636.png)



#### Saga不保证隔离性

- 语义锁
- 更多办法参考《微服务架构设计模式》，包括Saga

![1608390060820](MicroserviceDistributedSystem.assets/1608390060820.png)



#### 相关概念

- Saga 引擎
- 微服务编排引擎（Orchestrator）/协调器（Coordinator）
- 工作流引擎
- 状态机引擎
- 分布式事务中间件



### 阿里分布式事务中间件Seata简析

#### Seata 背景

- https://github.com/seata/seata
- 阿里巴巴分布式事务
- 蚂蚁金服分布式事务
- 前身是 fescar

![1608429730178](MicroserviceDistributedSystem.assets/1608429730178.png)



#### Seata 概念

- 一个全局分布式事务是由若干个本地分支事务组成

![1608430697997](MicroserviceDistributedSystem.assets/1608430697997.png)





#### Seata 角色

- Transaction Coordinator（TC）
- Transaction Manager（TM）
- Resource Manager（RM）

![1608430734600](MicroserviceDistributedSystem.assets/1608430734600.png)





#### Seata 原理

- XID，在TC中，确定是哪一个分支的事务
- TC 需要保证高可用

![1608430835177](MicroserviceDistributedSystem.assets/1608430835177.png)



#### Seata 事务注解

- Spring 的注解 @Transactional
- Seata 的 @GlobalTransactional
- Seata 的 全局锁机制，@GlobalLock，支持轻量级全局锁定隔离

![1608430918750](MicroserviceDistributedSystem.assets/1608430918750.png)



#### Seata-AT 原理：Phase 1

- 两阶段的第一阶段，SQL事务已经提交

![1608430989723](MicroserviceDistributedSystem.assets/1608430989723.png)



#### Seata-AT 原理：Phase 2 Commit

- 成功提交，找到 Undo log，删除
- 可以异步执行

![1608431010046](MicroserviceDistributedSystem.assets/1608431010046.png)



#### Seata-AT 原理：Phase 2 Rollback

- 失败回滚
- 找到回滚 SQL，执行，并提交回滚
- 可以异步执行

![1608431036331](MicroserviceDistributedSystem.assets/1608431036331.png)



#### Sata 通用分布式事务框架

- 推荐使用 AT Mode

![1608431171494](MicroserviceDistributedSystem.assets/1608431171494.png)



#### Sata 高可用部署

- 部署多台 TC
- 提供注册中心，配置中心
- 基于 Seata 解决微服务架构下数据一致性的实践：https://github.com/seata/seata-samples/tree/master/ha

![1608431357062](MicroserviceDistributedSystem.assets/1608431357062.png)



#### 支持框架、数据库和模式

- RPC
  - Dubbo
  - Spring Cloud
  - Motan
  - SPFA-RPC
  - 自定义RPC框架
- 数据库
  - MySQL，Oracle，postgreSQL，TiDB和RDS系列等
  - AT 模式
- 模式
  - AT（推荐）
  - TCC
  - Saga ~ 异步长事务





#### 参考资料

- Seata PPT 文档
- https://github.com/seata/awesome-seata



### Uber 微服务编排引擎Cadence简析

#### Uber Cadence 背景

- https://github.com/uber/cadence
- Cadence is a distributed, scalable, durable, and highly available orchestration engine to execute asynchronous long-running business logic in a scalable and resilient way.
- https://cadenceworkflow.io/



#### Uber Cadence 主要作者

![1608433326234](MicroserviceDistributedSystem.assets/1608433326234.png)



#### Uber 案例 ~ 小费Tips

- 客户扣钱，司机加钱
- 两次调用保证事务性

![1608433381927](MicroserviceDistributedSystem.assets/1608433381927.png)



#### Tips with Cadence

- Cadence 是一个工作流引擎
- Activity Worker，两个小的事务
- Workflow Worker，驱动全局事务的执行
- Workflow Starter，显示的启用

![1608433475037](MicroserviceDistributedSystem.assets/1608433475037.png)



#### Cadence 编程模式

- Activity
  - 活动，任务Task，处理器 Handler，微服务，Actor，分支事务执行者 
- Workflow
  - 工作流，编排流程，分布式事务流程
  - 长短事务都支持
- Starter
  - 启动，发信号，查询
  - 同步，异步都支持



#### Cadence Activity 功能

- 支持运行任何应用程序代码
- 支持长期运行任务(heartbeating)
- 支持异步执行
- 根据设置的重试策略进行自动重试
- 支持路由到指定的主机或者进程
- 通过队列派遣任务执行
- 支持对worker进行限流
- 支持对queue进行限流



#### Cadence Activity 样例

![1608433784332](MicroserviceDistributedSystem.assets/1608433784332.png)



#### Cadence Workflow 功能

- 支持虚拟对象技术（Virtual Objects）
- 支持事务
- 对活动(Activities)进行编排
- 支持接收外部事件并作出响应
- 有状态(包含局部变量和栈)
- 支持长期运行
- 支持持久化的时钟



#### Cadence Workflow 样例

![1608433872244](MicroserviceDistributedSystem.assets/1608433872244.png)



#### Activity 重试

![1608433941405](MicroserviceDistributedSystem.assets/1608433941405.png)



#### 事务补偿

![1608433982083](MicroserviceDistributedSystem.assets/1608433982083.png)



#### Cadence 客户端库

- java & go
- 无状态
- 屏蔽开发复杂性
- 支持以容错Actor方式开发分布式业务逻辑
  - Distributed business logic programmed as fault tolerant actors 



#### Cadence Web UI ~ 工作流查询

- https://github.com/uber/cadence-web

![1608434104858](MicroserviceDistributedSystem.assets/1608434104858.png)



#### Cadence Web UI ~ 工作流执行细节

- https://github.com/uber/cadence-web
- 类似于 java 语言执行调用栈

![1608434143104](MicroserviceDistributedSystem.assets/1608434143104.png)



#### Cadence 实现 Saga模式

- https://github.com/uber/cadence-java-samples/tree/master/src/main/java/com/uber/cadence/samples/bookingsaga
- 添加补偿，saga.addCompensation()

![1608434199474](MicroserviceDistributedSystem.assets/1608434199474.png)



#### 参考资料

- Cadence: The Only Workflow Platform You'll Ever Need. 
  -  https://www.youtube.com/watch?v=llmsBGKOuWl&t=606sCadence 
- Meetup: Introduction to Cadence
  -  长事务案例UberEATS(送餐服务)
  - https://www.youtube.com/watch?v=-Bulkhlc-RMuber 
- Cadence: Fault Tolerant Actor Framework
  - 长事务案例uDebitReward(司机积分服务)
  - https://www.youtube.com/watch?v=qce_AqCkFys&t=197s
- 更多同步/异步执行样例
  - https://github.com/uber/cadence-java-samples





### 如何理解 Uber Cadence 的架构设计？

#### Cadence 微服务架构

- Matching Service
- History Service
- Cassandra，MySQL

![1608438003738](MicroserviceDistributedSystem.assets/1608438003738.png)



#### Frontend Service

- 相当于一个BFF或者Facade服务
- 大部分透传调用History和Matching Service
- 暴露对全局实体的CRUD操作
- 暴露可视化(Visibility)API



#### Frontend 

- History Service 和 Matching Service 是有状态的
- Ubuer Ringpop：https://github.com/uber/ringpop-go

![1608438648718](MicroserviceDistributedSystem.assets/1608438648718.png)



#### History Service

- 两个队列 Transfer queue 和 Timer queue
- Shard Controller

![1608438712363](MicroserviceDistributedSystem.assets/1608438712363.png)



#### 如何绑定分片（Shard）？

![1608438775680](MicroserviceDistributedSystem.assets/1608438775680.png)

![1608438791058](MicroserviceDistributedSystem.assets/1608438791058.png)



#### Shard RangeID

- Acqure Shard 操作，接着原子操作（CAS） writes，对 range_id +1操作
- 当另一个 Host 加入的时候，继续 Acqure Shard 操作，range_id +1操作
- 原来的 Host 就不能继续 Shard

![1608438828818](MicroserviceDistributedSystem.assets/1608438828818.png)

![1608438895863](MicroserviceDistributedSystem.assets/1608438895863.png)

![1608438944309](MicroserviceDistributedSystem.assets/1608438944309.png)

![1608438958024](MicroserviceDistributedSystem.assets/1608438958024.png)

![1608438977266](MicroserviceDistributedSystem.assets/1608438977266.png)



#### Cadence 为什么要支持事务性分发？

![1608439048195](MicroserviceDistributedSystem.assets/1608439048195.png)



#### Transfer queue

- 实现事务的流转

![1608439072903](MicroserviceDistributedSystem.assets/1608439072903.png)

![1608439099974](MicroserviceDistributedSystem.assets/1608439099974.png)

![1608439118247](MicroserviceDistributedSystem.assets/1608439118247.png)



#### 关键技术 ~ 事务性发件箱

- 事务性发件箱是事务一致性分发的解决方案

![1608439227845](MicroserviceDistributedSystem.assets/1608439227845.png)



#### Cadence 中的分布式问题和技术

- 任务处理器
  - Handler 
  - Actor
  - 微服务
- 发现和路由(Discovery & Routing)
- 负载均衡(Load Balancing)
- 分布式存储
- 分片Shardinng
- 队列机制Queuing
- 事件溯源(Event Sourcing)
- 组成员协议+一致性Hash(Ringpop)
- 持久化Timer(延迟任务)
- 限流和流控



#### 参考资料

- Cadence Meetup: Cadence Architecture
- https://www.youtube.com/watch?v=5M5eiNBUf4Q
- Cadence源码(可以从早期release看起)
- https://github.com/uber/cadence
- 阅读源码：cadence-v0.1.0-beta
- https://github.com/uber/cadence/releases/tag/v0.1.0-beta



### 如何实现遗留系统的解耦拆分？

#### 什么时候该解耦拆分？

- 研发速度(Velocity)
  - 单体耦合系统造成交付效率低下
  - 团队间相互依赖，无法独立交付
  - 新人上手产出很慢
- 扩展(Scaling)
  - 无法再对单体系统进行垂直(vertical)扩展
  - 系统的某些部分需要独立扩展能力
- 部署(Deployment)
  - 系统的某些部分需要独立部署的能力
  - 单体发布太慢、太复杂、风险太高

![1608455897847](MicroserviceDistributedSystem.assets/1608455897847.png)



#### 共享单体 DB ~ StichFix 案例

- GeeCON 2019: Randy Shoup - Scaling yout Architecture with Services and Events
- https://www.youtube.com/watch?v=zVJSTMD6zg0

![1608456114118](MicroserviceDistributedSystem.assets/1608456114118.png)



#### 解耦拆分案例场景

![1608456152527](MicroserviceDistributedSystem.assets/1608456152527.png)



#### 步骤1：创建一个新服务

![1608456186383](MicroserviceDistributedSystem.assets/1608456186383.png)



#### 步骤2：应用走服务调用

![1608456220538](MicroserviceDistributedSystem.assets/1608456220538.png)



#### 不要停在步骤2

- 所有分布式系统的问题
- 所有共享DB的问题
- 没有获得微服务的任何好处

![1608456330065](MicroserviceDistributedSystem.assets/1608456330065.png)



#### 步骤3：迁移到独立DB

![1608456359118](MicroserviceDistributedSystem.assets/1608456359118.png)



#### 步骤4：继续拆分服务

![1608456395552](MicroserviceDistributedSystem.assets/1608456395552.png)



#### 步骤5：继续拆分服务

![1608456423612](MicroserviceDistributedSystem.assets/1608456423612.png)





#### 完成

- 基础领域服务，独立数据源

![1608456479309](MicroserviceDistributedSystem.assets/1608456479309.png)





### 拍拍贷系统拆分项目案例

#### 拆分前

问题：

- 所有数据在同一台数据库服务器上
- 应用可以不通过接口直接访问数据
- 跨库join严重，单点问题
- CPU时常飙高
- 磁盘空间吃紧
- QPS > 4w+(正常1w左右)

![1608457254391](MicroserviceDistributedSystem.assets/1608457254391.png)

![1608457298940](MicroserviceDistributedSystem.assets/1608457298940.png)



#### 拆分后

- 所有应用访问数据库，全部通过服务接口 api

![1608457341877](MicroserviceDistributedSystem.assets/1608457341877.png)



#### 拆分技术1 ~ 接口收口

![1608457356933](MicroserviceDistributedSystem.assets/1608457356933.png)



#### 收口沟通

![1608457377826](MicroserviceDistributedSystem.assets/1608457377826.png)



#### 拆分技术2 ~ 数据库迁移

- 写新DB，读写老DB，数据补偿
- 读写新DB，读写老DB，数据对比
- 读写新DB，写老DB
- 读写新DB，不操作老DB
- 使用 Apollo 配置中心开关

![1608457398214](MicroserviceDistributedSystem.assets/1608457398214.png)



#### DB 监控

![1608457425578](MicroserviceDistributedSystem.assets/1608457425578.png)



#### 迁移技术3 ~ 数据分发去 Join

- 服务层双写
- 分发器，将数据推到相关的业务库中
- MQ，是 消息队列PMQ

![1608457474463](MicroserviceDistributedSystem.assets/1608457474463.png)



#### 拆分迁移关键步骤小节

- 接口收口
- 双写迁移 DB
  - 增量可回滚
  - 数据补偿+对比
- 数据分发去 Join
- 计划 + 梳理 + 沟通  + 监控



### CQRS / CDC 技术在 Netflix 的实践

#### 数据同步和填充平台 Delta

- An eventual consistent, event driven, data synchronization and enrichment platform.
- 基于CDC/CQRS模式
- 场景需求
  - 搜索引擎
  - 数据仓库
  - 事件驱动工作流
- 现有解决办法的问题
  - 双写
    - 需要后台修复程序才能保持一致性
    - 延迟+开销
  - 事务性发件箱+Polling
    - 多语言环境，难以提供标准库
    - 应用侵入性
  - 分布式事务XA
    - 性能和死锁问题
    - 某些系统如ES不支持XA



#### 使用  Delta 之前的 Movice Search

- 查询数据 Movie Service DB
- 数据填充，Deal Service，Talent Service ......

![1608458387863](MicroserviceDistributedSystem.assets/1608458387863.png)



#### 使用  Delta 之后的 Movice Search

- 使用CDC，捕获数据库的变更记录

![1608458411208](MicroserviceDistributedSystem.assets/1608458411208.png)



#### 整体架构

![1608458437890](MicroserviceDistributedSystem.assets/1608458437890.png)



#### DB Log

- CDC 自研产品
- 功能亮点
  - 数据顺序性
    - 全量或部分Dump
    - CDC和Dump可以交替进行
    - 不锁表
    - 各种输出源
    - 高可用部署

![1608458512454](MicroserviceDistributedSystem.assets/1608458512454.png)



#### 参考资料

- Delta: A Data Synchronization and Enrichment Platform
- https://netflixtechblog.com/delta-a-data-synchronization-and-enrichment-platform-e82c36a79aee
- DBLog:A Generic Change-Data-Capture Framework
- https://netflixtechblog.com/dblog-a-generic-change-data-capture-framework-69351fb9099b
- Capturing Data Evolution in a Service Oriented Architecture
- https://medium.com/airbnb-engineering/capturing-data-evolution-in-a-service-oriented-architecture-72f7c643ee6f



### 小结

#### 微服务的四大技术难题

![1608365934145](MicroserviceDistributedSystem.assets/1608365934145.png)



#### 如何解决微服务的数据一致性分发问题？

- 事务性发件箱模式(Transactional Outbox)
- 变更数据捕获(Change Data Capture,CDC)
- 双写?
  - 需要后台校验补偿
- 确保单一真实数据源(Single Source of Truth)

![1608373971899](MicroserviceDistributedSystem.assets/1608373971899.png)



#### 如何解决微服务的数据聚合Join问题？

- BFF聚合层
- 反正规化+流聚合
  - 分布式物化视图(Materialized View)
  - CQRS模式
- CQRS最终一致性
  - UI更新

![1608376361264](MicroserviceDistributedSystem.assets/1608376361264.png)



#### 如何解决微服务的分布式事务问题？

- 2PC/XA/TCC
- Saga模式
  - 协同VS编排
  - 需考虑隔离性
- 主流开源产品
  - 2PC/XA/TCC方案~ Seata
  - 工作流/编排/Saga方案~ Cadence

![1608430835177](MicroserviceDistributedSystem.assets/1608430835177.png)

![1608433475037](MicroserviceDistributedSystem.assets/1608433475037.png)



#### 如何实现遗留系统的解耦拆分？

- 接口收口
- 拆分迁移DB
  - 双写＋开关,增量可回滚
  - 数据补偿＋比对
- 数据分发去Join

![1608456423612](MicroserviceDistributedSystem.assets/1608456423612.png)



## 如何设计一个高并发无状态的会话缓存服务 - 携程SessionServer案例 

### SessionServer 项目背景和目标 

#### 粘性会话（Sticky Session）101

- Session会话机制，保存用户状态信息
- 粘性会话 通过 LB，将用户会话绑定到 对应的应用服务器

![1608461908579](MicroserviceDistributedSystem.assets/1608461908579.png)



#### 粘性会话的问题

- LB  负载相对不均衡
- 应用和负载均衡设备耦合
  - 一道架构的枷锁，任何调整都要考虑到它
- 应用有状态
  - 单点问题
  - 发布问题
  - 难以水平扩展
    - Scale Out
    - 跨数据中心 HA



#### 负载不均衡问题

- Session 请求不均匀，有状态请求

- API  请求监控均衡，无状态请求

![1608461996997](MicroserviceDistributedSystem.assets/1608461996997.png)



#### 诡异的客户慢问题

- 应用服务器有两台慢，刚好用户Session绑定到这两台服务器上
- 由于 LB 的不均衡，导致服务器请求大，资源不足

![1608462013959](MicroserviceDistributedSystem.assets/1608462013959.png)



#### 面试题 ~ 四种常用会话技术

- 粘性会话
- 纯客户端会话（浏览器的Cookie大小限制 4K）
- 服务器共享会话
- 服务器端集中式会话（集中存储）

![1608462079105](MicroserviceDistributedSystem.assets/1608462079105.png)



#### 携程当时现状（2014年初）

- 10000+服务器
- 2000+App
- 80,000,000PV
- Ajax >>PV
- session大小?1K 5K 50K 100K 1M ...
- 20分钟同时在线:6,000,000
- web应用主要基于ASP.Net开发
- https://www.ctrip.com/



#### SessionServer 架构和设计目标

- 消除Sticky Session，支持应用 Scale out（横向扩展）
- 高并发
- 高性能（99.99%<10ms)
- 高可用（HA)
- 水平按需扩展（直接添加服务器就可以实现扩展）
  - 透明扩容
- 透明升级（升级更新SessionServer）
- 支持跨数据中心（同城双活）
- 接入简单（不用改代码)
- 监控和运维友好（全部的监控）



### 总体架构设计

#### Thrift IDL

- ASP.NET state server Protocol
- https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-asp/83f6b453-c695-419c-998b-0aa50279bc40
- Thrift  二进制的通讯协议

![1608473690052](MicroserviceDistributedSystem.assets/1608473690052.png)



#### 关键设计问题

- SessionID和SessionSever映射关系?
  - 分布式 KV Store
  -  一致性Hash? Raft?
- SessionServer挂了怎么办?
- 升级扩容?
- 服务发现?

![1608473928547](MicroserviceDistributedSystem.assets/1608473928547.png)



#### SessionServer  总体架构

- 将 SessionServer  的 IP 进行来回传递，记录在浏览器的Cookie中
- SessionServer  将 Session数据，异步写到数据库
- SessionServer 出问题，Session client 会检测到，选择另一个 SessionServer  完成服务

![1608473961431](MicroserviceDistributedSystem.assets/1608473961431.png)



#### SessionServer 内部设计

- 将热集的Session，存储在内存中
- 根据最近最少访问原则，将Session踢出到二级缓存

![1608474014121](MicroserviceDistributedSystem.assets/1608474014121.png)



#### Workflow ~ 第一次请求处理

![1608474033706](MicroserviceDistributedSystem.assets/1608474033706.png)



#### Workflow ~ 后续读请求

![1608474071263](MicroserviceDistributedSystem.assets/1608474071263.png)



#### Workflow ~ 后续写请求

![1608474095326](MicroserviceDistributedSystem.assets/1608474095326.png)



#### Workflow ~ 后续读失败

![1608474145178](MicroserviceDistributedSystem.assets/1608474145178.png)



#### Workflow ~ 后续写失败

![1608474158418](MicroserviceDistributedSystem.assets/1608474158418.png)



#### 跨数据中心 HA ~ 摇摆策略

- 两个数据中心互通
- 使用 euraka 实现，数据中心之间可以相互通信
- 保证请求到达任何一个数据中心，都可以找到对应的session

![1608474225742](MicroserviceDistributedSystem.assets/1608474225742.png)



#### 跨数据中心 HA ~ 双写策略

- 在 Session 写入的时候，同时写到本地的 SessionServer和另一个数据中心的 SessionServer
- 使用 Memcached 实现缓存
- https://github.com/Netflix/EVCache

![1608474274916](MicroserviceDistributedSystem.assets/1608474274916.png)



#### 优化和 x-pipe

- 后端存储使用缓存（redis）实现，性能优于 MySQL
- 携程 x-pipe，基于 Master-Slave 协议，实现跨数据中心同步数据
- https://github.com/ctripcorp/x-pipe

![1608474327970](MicroserviceDistributedSystem.assets/1608474327970.png)





### 如何设计一个高性能基于内存的 LRU Cache

#### 术语

| Cache Hit      | 缓存命中     |
| -------------- | ------------ |
| Cache Miss     | 缓存不命中   |
| Cache Capacity | 缓存容量     |
| LRU            | 最近最少使用 |
| LFU            | 最不经常使用 |
| Expiration     | 缓存过期     |
| Eviction       | 缓存剔除     |
| Purge          | 缓存清理     |
| Activate       | 缓存激活加载 |
| Write Behind   | 写后         |
| Write Through  | 写穿透       |



![1608474014121](MicroserviceDistributedSystem.assets/1608474014121.png)



#### 面试题：如何设计一个LRU 缓存 V1

##### LRU Cache 原理

- 头结点就是下一个被剔除的对象
- 每一次查询的对象，都被移动到尾结点
- 当对象的大小超过总体的容量的时候，清理最近最少使用的对象，也即头结点

![1608511054949](MicroserviceDistributedSystem.assets/1608511054949.png)



##### LRU Cache 实现 V1

- Node
- removeNode()
- offerNode()
- put()
- get()
- https://github.com/spring2go/okcache/tree/master/src/main/java/com/spring2go/lrucache/v1

```java
import java.util.HashMap;
import java.util.Map;

/**
 * LruCache实现，V1版本，线程不安全
 * <p>
 * Created on Jul, 2020 by @author bobo
 */
public class LruCacheV1<K, V> {

    private int maxCapacity;
    private Map<K, Node<K, V>> map;
    private Node<K, V> head, tail;

    private static class Node<K, V> {
        private V value;
        private K key;
        private Node<K, V> next, prev;

        public Node(K key, V value) {
            this.key = key;
            this.value = value;
        }

        @Override
        public String toString() {
            return value.toString();
        }
    }

    // 从双向链表中移除一个节点
    private void removeNode(Node<K, V> node) {
        if (node == null) return;

        if (node.prev != null) {
            node.prev.next = node.next;
        } else {
            head = node.next;
        }

        if (node.next != null) {
            node.next.prev = node.prev;
        } else {
            tail = node.prev;
        }
    }

    // 向双向链表的尾部添加一个节点
    private void offerNode(Node<K, V> node) {
        if (node == null) return;

        if (head == null) {
            head = tail = node;
        } else {
            tail.next = node;
            node.prev = tail;
            node.next = null;
            tail = node;
        }
    }

    public LruCacheV1(final int maxCapacity) {
        this.maxCapacity = maxCapacity;
        map = new HashMap<>();
    }

    public void put(K key, V value) {
        if (map.containsKey(key)) {
            Node<K, V> node = map.get(key);
            node.value = value;
            removeNode(node);
            offerNode(node);
        } else {
            if (map.size() == maxCapacity) {
                map.remove(head.key);
                removeNode(head);
            }
            Node<K, V> node = new Node<>(key, value);
            offerNode(node);
            map.put(key, node);
        }
    }

    public V get(K key) {
        Node<K, V> node = map.get(key);
        if (node == null) return null;
        removeNode(node);
        offerNode(node);
        return node.value;
    }

    public int size() {
        return map.size();
    }
}
```





#### 面试题：如何设计一个线程安全的LRU 缓存 V2

##### LRU Cache 实现 V2

- ReentrantReadWriteLock()
- https://github.com/spring2go/okcache/tree/master/src/main/java/com/spring2go/lrucache/v2

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * LruCache实现，V1版本，线程安全
 *
 * Created on Jul, 2020 by @author bobo
 */
public class LruCacheV2<K, V> {
    private int maxCapacity;
    private Map<K, Node<K, V>> map;
    private Node<K, V> head, tail;

    private ReadWriteLock lock = new ReentrantReadWriteLock();
    private Lock writeLock = lock.writeLock();
    private Lock readLock = lock.readLock();

    private static class Node<K, V> {
        private V value;
        private K key;
        private Node<K, V> next, prev;

        public Node(K key, V value) {
            this.key = key;
            this.value = value;
        }

        @Override
        public String toString() {
            return value.toString();
        }
    }

    // 从双向链表中移除一个节点
    private void removeNode(Node<K, V> node) {
        if (node == null) return;

        if (node.prev != null) {
            node.prev.next = node.next;
        } else {
            head = node.next;
        }

        if (node.next != null) {
            node.next.prev = node.prev;
        } else {
            tail = node.prev;
        }
    }

    // 向双向链表的尾部添加一个节点
    private void offerNode(Node<K, V> node) {
        if (node == null) return;

        if (head == null) {
            head = tail = node;
        } else {
            tail.next = node;
            node.prev = tail;
            node.next = null;
            tail = node;
        }
    }

    public LruCacheV2(final int maxCapacity) {
        this.maxCapacity = maxCapacity;
        map = new HashMap<>();
    }

    public void put(K key, V value) {
        writeLock.lock();
        try {
            if (map.containsKey(key)) {
                Node<K, V> node = map.get(key);
                node.value = value;
                removeNode(node);
                offerNode(node);
            } else {
                if (map.size() >= maxCapacity) {
                    map.remove(head.key);
                    removeNode(head);
                }
                Node<K, V> node = new Node<>(key, value);
                offerNode(node);
                map.put(key, node);
            }
        } finally {
            writeLock.unlock();
        }
    }

    public V get(K key) {
        writeLock.lock();
        try {
            Node<K, V> node = map.get(key);
            if (node == null) return null;
            removeNode(node);
            offerNode(node);
            return node.value;
        } finally {
            writeLock.unlock();
        }
    }

    public int size() {
        readLock.lock();
        try {
            return map.size();
        } finally {
            readLock.unlock();
        }
    }
}
```





#### 面试题：如何设计一个线程安全和高并发的LRU 缓存 V3

##### LRU Cache 实现 V3

- concurrentHashmap的两阶段分段锁机制
- 类似于分片 Shareding 操作
- cacheSegments
- https://github.com/spring2go/okcache/tree/master/src/main/java/com/spring2go/lrucache/v3

```java
/**
 *  LruCache实现，V3版本，线程安全+高并发
 *
 * Created on Jul, 2020 by @author bobo
 */
public class LruCacheV3<K, V> {

    private LruCacheV2<K, V>[] cacheSegments;

    public LruCacheV3(final int maxCapacity) {
        int cores = Runtime.getRuntime().availableProcessors();
        int concurrency = cores < 2 ? 2 : cores;
        cacheSegments = new LruCacheV2[concurrency];
        int segmentCapacity = maxCapacity / concurrency;
        if (maxCapacity % concurrency == 1) segmentCapacity++;
        for (int index = 0; index < cacheSegments.length; index++) {
            cacheSegments[index] = new LruCacheV2<>(segmentCapacity);
        }
    }

    public LruCacheV3(final int concurrency, final int maxCapacity) {
        cacheSegments = new LruCacheV2[concurrency];
        int segmentCapacity = maxCapacity / concurrency;
        if (maxCapacity % concurrency == 1) segmentCapacity++;
        for (int index = 0; index < cacheSegments.length; index++) {
            cacheSegments[index] = new LruCacheV2<>(segmentCapacity);
        }
    }

    private int segmentIndex(K key) {
        int hashCode = Math.abs(key.hashCode() * 31);
        return hashCode % cacheSegments.length;
    }

    private LruCacheV2<K, V> cache(K key) {
        return cacheSegments[segmentIndex(key)];
    }

    public void put(K key, V value) {
        cache(key).put(key, value);
    }

    public V get(K key) {
        return cache(key).get(key);
    }

    public int size() {
        int size = 0;
        for (LruCacheV2<K, V> cache : cacheSegments) {
            size += cache.size();
        }
        return size;
    }
}
```





#### SessionServer LRU Cache  ~ okCache

- 基于Guava Cache改造简化
- 核心思想类似LRUCache实现V3
  - 双端队列Deque
  - 分段锁+锁优化
- 支持过期清除
- 支持超过容量剔除到本地可持久化缓存（BigCache)
- 剔除由get/put操作触发，无需后台线程
- 支持写回到后端持久化存储（DB)
- 埋点统计
- https://github.com/spring2go/okcache/tree/master/src/main/java/com/spring2go/okcache



#### Caffeine （生产推荐）

- Guava Cache的升级版
- window TinyLFU
- 高并发RingBuffer
- https://github.com/ben-manes/caffeine

![1608511391956](MicroserviceDistributedSystem.assets/1608511391956.png)





### 如何设计一个高性能大容量持久化的ConcurrentHashmap

#### 会话缓存溢出问题

- 二级缓存只追求容量
- 当内存缓存达到内存的最大容量的时候，就需要进行会话缓存的移出，到二级缓存

![1608474014121](MicroserviceDistributedSystem.assets/1608474014121.png)



#### 可持久化的ConcurrentHashmap ~ BigCache

- Key常驻内存（Key不大），Value可持久化（Value大）
  - 分段读写锁
- 读写存储块，定制GC
- 支持三种模式
  - 纯磁盘文件模式
  - 内存映射+磁盘文件
  - 堆外内存+磁盘文件
- 埋点统计
- 更新操作，删除多了，会有存储空洞，垃圾碎片，Cleaning Threads 实现清理
- https://github.com/spring2go/okcache/tree/master/src/main/java/com/spring2go/bigcache

![1608511771891](MicroserviceDistributedSystem.assets/1608511771891.png)



#### Yahoo HaloDB

- Key常驻内存，Value持久化
  - 小Key (8字节)，大Value（平均10KB）
  - 总体存储量>内存
- 高并发读写
- 高性能低延迟
- Append Only（只添加不直接删除，标记删除），墓碑删除机制
- 后台线程定期Compact
- https://github.com/yahoo/HaloDB

![1608511942508](MicroserviceDistributedSystem.assets/1608511942508.png)



### 设计评估和总结

#### 设计评估

| 1    | 消除 Stick Session，支持应用 Scale Out | 达成                                                         |
| ---- | -------------------------------------- | ------------------------------------------------------------ |
| 2    | 高并发                                 | 分段锁，SessionServer 集群分摊负载                           |
| 3    | 高性能                                 | 99百分位<10ms                                                |
| 4    | 高可用                                 | SessionServer挂一台或几台都不影响，分组 Group 隔离，扩数据中心 IDC |
| 5    | 水平按需扩展/透明扩展                  | 可以按需添加 sessionServer                                   |
| 6    | 透明升级                               | 可以按需对SessionServer进行上下线，对用户无影响              |
| 7    | 支持跨数据中心                         | 支持                                                         |
| 8    | 接入简单                               | 只需引用 SessionClient DLL，可以通过发布工具自动化           |
| 9    | 监控运维友好                           | 细粒度埋点和监控                                             |

![1608473961431](MicroserviceDistributedSystem.assets/1608473961431.png)



#### Hystrix 限流容错

![1608512298850](MicroserviceDistributedSystem.assets/1608512298850.png)



#### 生产性能监控

- 爬虫引发多次出发限流熔断监控

![1608512316588](MicroserviceDistributedSystem.assets/1608512316588.png)



#### 最佳实践

- 缓存是高并发/高性能的基础
  - 分段锁
  - LRU+二级缓存机制
- 高性能vs复杂度/可理解性的平衡
- “无状态”分布式缓存
  - 客户端cookie保存SessionServer lP
  - 写后( write Behind）到后端存储（状态备份)
- 分组隔离
- 增量应用迁移
- 透明扩容/升级
- 组件积木式开发
- 监控埋点+容错限流
- 单元/并发和性能测试



## 系统设计综合案例 - SaaS服务healthchecks.io的设计 

### SaaS 项目 healthchecks.io的背景和架构

#### healthchecks.io

- 定时任务( Cron Job)监控
- 开源Saas 项目
  -  Python & Django
- 一个人的SaaS产品
- https://healthchecks.io/
- https://github.com/healthchecks



#### 定时任务监控原理

1. 用户通过healthchecks.io(HC)设置监控目标的定时任务Cron表达式 
2. HC监控来自被监控服务的ping请求
3. 如果ping 准时到达，那么HC保持静默
4. 如果ping未准时到达，那么HC就发出告警

![1609213008620](MicroserviceDistributedSystem.assets/1609213008620.png)

![1609213020060](MicroserviceDistributedSystem.assets/1609213020060.png)



#### 应用场景

定时任务(Cron Jobs)

1. 文件系统备份
2. 数据库备份
3. 定期业务数据导入
4. 定期数据清理
5. 定期病毒扫描
6. SSL过期更新

进程、服务、服务器

1. 定期检查应用进程存活
2. 队列堆积监控
3. 定期检查机器/VM/容器存活
4. 系统资源监控（磁盘/内存...)



#### UI - 监控面板

![1609213163566](MicroserviceDistributedSystem.assets/1609213163566.png)



#### UI -  Check 配置

![1609213202267](MicroserviceDistributedSystem.assets/1609213202267.png)



#### 告警集成

![1609213231728](MicroserviceDistributedSystem.assets/1609213231728.png)



#### 一个人的SaaS产品

- https://blog.healthchecks.io/2019/08/healthchecks-pycon-lithuania-19/

![1609213307796](MicroserviceDistributedSystem.assets/1609213307796.png)



#### 业务状况

![1609213334859](MicroserviceDistributedSystem.assets/1609213334859.png)



#### 架构 2016

![1609213358841](MicroserviceDistributedSystem.assets/1609213358841.png)



#### 架构 2017

![1609213378043](MicroserviceDistributedSystem.assets/1609213378043.png)



#### 如何设计 myhc.io?

架构设计需求

1. 高性能/高可用/可扩展
2. 核心领域模型
3. 延迟任务
4. 锁
5. 限流
6. 防攻击/防爬虫



#### 核心领域模型（不包括支付）

- https://github.com/healthchecks/healthchecks/blob/master/hc/api/models.py

![1609213500828](MicroserviceDistributedSystem.assets/1609213500828.png)



#### myhc.io 的架构

![1609213549131](MicroserviceDistributedSystem.assets/1609213549131.png)



#### 参考

- Healthchecks.io 博客
  - https://blog.healthchecks.io/
- 开源异常日志健康 SaaS 平台 Sentry
  - https://github.com/getsentry



### 如何设计一个轻量级的延迟任务队列

#### Check Model

- code
- schedule
- last_ping
- alert_after
- status

![1609213689398](MicroserviceDistributedSystem.assets/1609213689398.png)



#### 轻量持久化的延迟任务

- https://github.com/healthchecks/healthchecks/blob/master/hc/api/management/commands/sendalerts.py
- 乐观锁

![1609213805616](MicroserviceDistributedSystem.assets/1609213805616.png)



#### Db-scheduler 项目

- 参考项目：集群持久化友好的调度器
- 本质也是一种延迟任务队列
- 支持周期和一次性延迟任务
- https://github.com/kagkarlsson/db-scheduler

![1609213866975](MicroserviceDistributedSystem.assets/1609213866975.png)



#### Killbill Notification Queue 项目

- 本质是一种延迟任务队列
- 支持一次性延迟任务

![1609214016877](MicroserviceDistributedSystem.assets/1609214016877.png)



#### Cron 表达式处理 项目

- 参考项目：java 的定时表达式工具
- https://github.com/jmrozanec/cron-utils

![1609214127526](MicroserviceDistributedSystem.assets/1609214127526.png)





#### 其他任务调度开源项目

- Quartz（java）
  - https://github.com/quartz-scheduler/quartz
- xxl-job（java）
  - https://github.com/xuxueli/xxl-job
- Celery（python）
  - https://github.com/celery/celery
- Hangfire（c#）
  - https://github.com/HangfireIO/Hangfire



### 如何设计一把轻量级的锁？

#### 锁的应用场景和类型

场景

- 周期性处理任务/唯─执行者
  - PMQ定期清理七天前的老消息
  - Healthchecks.io
    - 定期Check/Alert
    - 定期清理ping数据
    - 定期扣费( billing)
- Master/Standby高可用(一个决策大脑)
  - K8s Scheduler/Mesos Master
  - PMQ的重平衡协调者
- 多个线程/进程/微服务更新共享数据场景
  - 微服务场景->分布式锁

类型

- 乐观锁
- 悲观锁





#### 乐观锁

- 基于一个版本号
- 默认写的成功性大远远大于失败的概率

![1609214472541](MicroserviceDistributedSystem.assets/1609214472541.png)



#### 悲观锁

- 基于一个全局锁的服务，掌控资源

![1609214488338](MicroserviceDistributedSystem.assets/1609214488338.png)



#### 栅栏令牌（fencing token）锁

- 使用令牌的方式，自增1
- 设置一个锁过期时间，防止长期占用锁

![1609214596682](MicroserviceDistributedSystem.assets/1609214596682.png)



#### ShedLock 项目

- 适用于周期任务
  - 唯一执行者场景
- 支持多种Lock Providers
  - Jdbc
  - Mongo
  - zookeeper
  - Redis
  - Hazelcast
- 支持锁超时( lockAtMostFor )
- 支持至少执行时间( lockAtLeastFor )
- https://github.com/lukas-krecan/ShedLock



### 如何设置一个分布式限流系统？

#### Github API Rate Limiting

![1609214886131](MicroserviceDistributedSystem.assets/1609214886131.png)



#### LinkedIn API Rate Limiting

![1609214923105](MicroserviceDistributedSystem.assets/1609214923105.png)



#### Bitly API Rate Limiting

![1609214950602](MicroserviceDistributedSystem.assets/1609214950602.png)



#### 如何设计一个分布式限流系统？

场景

1. 防止恶意访问(DDoS)
2. 用户程序问题
3. 支持双十一
4. 内部服务出问题
5. 服务本身有处理容量限制

架构设计需求

1. 高并发/高性能/高可用/可扩展
2. 最小化对用户请求的延迟
3. 限流实时和准确性
   1. 强一致vs最终一致
   2. 精确vs大致准确
4. 最小化内存占用



#### 令牌桶 or 漏桶算法

- 分发令牌，会导致请求不均匀，不平滑

![1609215135870](MicroserviceDistributedSystem.assets/1609215135870.png)



#### 令牌桶（每分钟填充1个令牌）

![1609215228691](MicroserviceDistributedSystem.assets/1609215228691.png)



#### 分布式的问题

- 同时操作数据的时候，get 和 set 之间的问题
- 如果不能保证操作原子性，那么限流就不准确

![1609215281595](MicroserviceDistributedSystem.assets/1609215281595.png)



#### 固定窗口计数器

- 固定一个时间段，多少请求，来限流

![1609215314077](MicroserviceDistributedSystem.assets/1609215314077.png)



#### 固定窗口计数器（每分钟限流2个请求）

![1609215373309](MicroserviceDistributedSystem.assets/1609215373309.png)



#### 固定窗口的问题

- 由于时间的窗口与窗口之间，会有一个边界
- 做不到流畅与平滑

![1609215458799](MicroserviceDistributedSystem.assets/1609215458799.png)



#### 滑动窗口计数器

- 保证限流请求的流畅性

![1609215477594](MicroserviceDistributedSystem.assets/1609215477594.png)



#### 滑动窗口计数器（每小时限流2个请求）

![1609215573075](MicroserviceDistributedSystem.assets/1609215573075.png)



#### Hystrix 滑动窗口计数器

- https://github.com/Netflix/Hystrix
- 每十秒为一个滑动窗口，作为一个 buckets

![1609215664529](MicroserviceDistributedSystem.assets/1609215664529.png)



#### 集中状态限流

- 使用一个限流服务，提供整体的限流

![1609215726168](MicroserviceDistributedSystem.assets/1609215726168.png)



#### 无状态限流

![1609215745479](MicroserviceDistributedSystem.assets/1609215745479.png)





#### 限流最佳实践

- 不要让限流器bug影响程序的正常工作
- 合理返回限流错误
  - HTTP 429 (Too Many Requests )
  - HTTP 503 ( Service Unavailable)
  - HTTP 200?（不让外界知道服务的状态，不管什么，都返回200）
- 可以按需关闭限流器
  - 功能开关
- 安全上线策略
  - 先打日志监控





#### 开源项目参考

- Bucket4j
  - 令牌桶算法，Github 800+ stars
  - https://github.com/vladimir-bukhtoyarov/bucket4j
- Ratelimiter4j
  - 固定窗口算法，Github 600+ stars
  - https://github.com/wangzheng0822/ratelimiter4j
- Ratelimitj
  - 滑动窗口算法，Github 300+ stars
  - https://github.com/mokies/ratelimitj
- Spring-cloud-zuul-ratelimit
  - zuul网关限流，Github 900+ stars
  - https://github.com/marcosbarbero/spring-cloud-zuul-ratelimit
- Sentinel
  - 中大规模企业级限流，Github 13k stars
  - https://github.com/alibaba/Sentinel

![1609215872138](MicroserviceDistributedSystem.assets/1609215872138.png)





### 如何设计一个分布式 TopK 系统实现防爬虫？

#### 如何设计一个分布式 TopK 系统？

场景

1. 防爬虫
2. 防DDoS攻击
3. 当前最热100微博/视频/新闻/商品/股票

功能需求
- topk (k, startTime, endTime)
- 分钟

架构设计需求

- 高性能
- 100ms内返回top 100列表
- 高可用
  - 不能有单点失败。
- 可扩展
  - 携程/B站规模
- 准确性
  - 尽可能准确



#### 单机方案

- 一般 十万级别的数据，可以使用单机实现
- 对一分钟的数据，进行排序，需要使用到算法
- 推荐，使用小堆排序

![1609216236637](MicroserviceDistributedSystem.assets/1609216236637.png)



#### TopK 算法实现

- min heap （最小堆）

![1609216375122](MicroserviceDistributedSystem.assets/1609216375122.png)



#### 多机分区方案

- 在分布式的场景下，需要进行多态主机进行分开聚合，然后归并排序

![1609216453301](MicroserviceDistributedSystem.assets/1609216453301.png)



#### 防爬虫总体架构设计（异步）

- 内部使用 Kafka 做为消息队列，使用 customer 进行日志的提取，将结果存入数据库
- 基于 TopK 算法，实现最近一段时间内 IP 访问最大值的检测

![1609216489931](MicroserviceDistributedSystem.assets/1609216489931.png)



## 如何实现精细化服务治理 - 服务网格技术ServiceMesh解析 

### 为什么说 ServiceMesh 是微服务的未来？

#### 微服务治理 - Must Have

- 服务发现
- 服务监控
- 弹性和容错

![1609292033126](MicroserviceDistributedSystem.assets/1609292033126.png)



#### 微服务治理 - Need To Hava

![1609292065621](MicroserviceDistributedSystem.assets/1609292065621.png)



#### 微服务治理演进史 - Dubbo(2012年前)

- 框架继承
- https://dubbo.apache.org/
- Dubbo 是借鉴 finagle 实现的

![1609292182682](MicroserviceDistributedSystem.assets/1609292182682.png)

![1609292237137](MicroserviceDistributedSystem.assets/1609292237137.png)



#### 微服务治理演进史 - Airbnb SmarkStack(2013)

- 主机独立进程
- 目前已经不维护项目了

![1609292322233](MicroserviceDistributedSystem.assets/1609292322233.png)

![1609292343947](MicroserviceDistributedSystem.assets/1609292343947.png)



#### 微服务治理演进史 - Netflix OSS/ Spring Cloud(2013年后)

- 框架组件
- Spring cloud 是在 Netflix 开源的基础上，分装出来的

![1609292427029](MicroserviceDistributedSystem.assets/1609292427029.png)

![1609292439420](MicroserviceDistributedSystem.assets/1609292439420.png)



#### 微服务治理演进史 - Envoy/ Istio(2016后)

- 编程（sidecar）or 服务网格（serviceMesh）
- https://istio.io/latest/docs/concepts/what-is-istio/

![1609292536093](MicroserviceDistributedSystem.assets/1609292536093.png)

![1609292567974](MicroserviceDistributedSystem.assets/1609292567974.png)



#### 比较

| 代   | 代表开源产品      | 年代     | 集成/部 署模式    | 优势                                      | 不足                                     | 支持组织                   |
| ---- | ----------------- | -------- | ----------------- | ----------------------------------------- | ---------------------------------------- | -------------------------- |
| 1    | Dubbo/Finagle     | 2012年前 | 框架继承          | 性能，代码层控制粒度                      | 多语言成本高，业务代码耦合，升级迁移麻烦 | Apache + 阿里              |
| 1.5  | Airbnb SmartStack | 2013     | 边车/主机独立进程 | 支持多语言，非侵入式                      | 部署运维成本高，非标准                   | Airbnb                     |
| 2    | Spring Cloud      | 2013年后 | 框架组件          | 性能，代码层控制粒度，组件化              | 多语言成本高，业务代码耦合，升级迁移麻烦 | Pivotal/Netflix            |
| 3    | Istio/Envoy       | 2016年后 | 边车/服务网格     | 支持多语言，非侵入，标准化，和k8s无缝集成 | 无法控制到业务代码层                     | 谷歌/IBM/Lyft/Redhat/Cisco |

![1609292967883](MicroserviceDistributedSystem.assets/1609292967883.png)



#### k8s + serviceMesh 是微服务的未来

- 数据中心虚拟化+跨横切面
- 关注点下沉，是行业总趋势

![1609293141029](MicroserviceDistributedSystem.assets/1609293141029.png)



### 解析 Envoy Proxy

#### Envoy 云原生代理

1. Out of process architecture
2. Modern C++11 code base
3. L3/L4 filter architecture
4. HTTP L4 filter architecture
5.  First class HTTP/2 support
6. HTTPL7 routing
7. gRPC support
8. Service discovery and dynamicconfiguration
9. Active/passive Health checking
10. Advanced load balancing
11. Service/middle/edge proxy
12. Best in class observability
13. Graceful restart
14. https://www.envoyproxy.io/docs/envoy/latest/intro/what_is_envoy

![1609293344511](MicroserviceDistributedSystem.assets/1609293344511.png)



#### 谁在用Envoy

![1609293369123](MicroserviceDistributedSystem.assets/1609293369123.png)



#### 核心概念

- Downstream
- Upstream

![1609293423739](MicroserviceDistributedSystem.assets/1609293423739.png)



#### 场景

1. 演示一个HTTP/2请求（TLS over TCP)
2. 唯一network filter ~HTTP connection manager
3. HTTP filter chain~一个CustomFilter + router filter
4. 文件系统访问日志
5. Statsd sink
6. 单个集群cluster
7. 静态配置
8. https://www.envoyproxy.io/docs/envoy/latest/intro/life_of_a_request



#### 静态配置

```yaml
static_resources:
  listeners:
  # There is a single listener bound to port 443.
  - name: listener_https
    address:
      socket_address:
        protocol: TCP
        address: 0.0.0.0
        port_value: 443
    # A single listener filter exists for TLS inspector.
    listener_filters:
    - name: "envoy.filters.listener.tls_inspector"
      typed_config: {}
    # On the listener, there is a single filter chain that matches SNI for acme.com.
    filter_chains:
    - filter_chain_match:
        # This will match the SNI extracted by the TLS Inspector filter.
        server_names: ["acme.com"]
      # Downstream TLS configuration.
      transport_socket:
        name: envoy.transport_sockets.tls
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.DownstreamTlsContext
          common_tls_context:
            tls_certificates:
            - certificate_chain: { filename: "certs/servercert.pem" }
              private_key: { filename: "certs/serverkey.pem" }
      filters:
      # The HTTP connection manager is the only network filter.
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          use_remote_address: true
          http2_protocol_options:
            max_concurrent_streams: 100
          # File system based access logging.
          access_log:
            - name: envoy.access_loggers.file
              typed_config:
                "@type": type.googleapis.com/envoy.extensions.access_loggers.file.v3.FileAccessLog
                path: "/var/log/envoy/access.log"
          # The route table, mapping /foo to some_service.
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["acme.com"]
              routes:
              - match:
                  path: "/foo"
                route:
                  cluster: some_service
          # CustomFilter and the HTTP router filter are the HTTP filter chain.
          http_filters:
            # - name: some.customer.filter
            - name: envoy.filters.http.router
  clusters:
  - name: some_service
    connect_timeout: 5s
    # Upstream TLS configuration.
    transport_socket:
      name: envoy.transport_sockets.tls
      typed_config:
        "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.UpstreamTlsContext
    load_assignment:
      cluster_name: some_service
      # Static endpoint assignment.
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: 10.1.2.10
                port_value: 10002
        - endpoint:
            address:
              socket_address:
                address: 10.1.2.11
                port_value: 10002
    typed_extension_protocol_options:
      envoy.extensions.upstreams.http.v3.HttpProtocolOptions:
        "@type": type.googleapis.com/envoy.extensions.upstreams.http.v3.HttpProtocolOptions
        explicit_http_config:
          http2_protocol_options:
            max_concurrent_streams: 100
  - name: some_statsd_sink
    connect_timeout: 5s
    # The rest of the configuration for statsd sink cluster.
# statsd sink.
stats_sinks:
  - name: envoy.stat_sinks.statsd
    typed_config:
      "@type": type.googleapis.com/envoy.config.metrics.v3.StatsdSink
      tcp_cluster_name: some_statsd_sink
```



#### 请求流程

#####  监听器接受TCP 连接

![1609293693401](MicroserviceDistributedSystem.assets/1609293693401.png)



##### 监听器和网络过滤链匹配

![1609293792233](MicroserviceDistributedSystem.assets/1609293792233.png)



##### TLS 传输层解密

![1609293833647](MicroserviceDistributedSystem.assets/1609293833647.png)



##### 网络过滤处理 + HTTP 2解码

![1609294055591](MicroserviceDistributedSystem.assets/1609294055591.png)



##### HTTP 过滤链处理

![1609294111620](MicroserviceDistributedSystem.assets/1609294111620.png)





##### 负载均衡

![1609294145736](MicroserviceDistributedSystem.assets/1609294145736.png)



##### HTTP 2 编码 + TLS传输层套接字加密

![1609294184278](MicroserviceDistributedSystem.assets/1609294184278.png)



##### 响应和事后处理

- 响应处理
- 提前终止
- 事后处理
  - 统计
  - 访问日志
  - 调用链 trace

![1609294255272](MicroserviceDistributedSystem.assets/1609294255272.png)





#### Envoy 的线程模型

- 内部支持请求事件循环

![1609294341296](MicroserviceDistributedSystem.assets/1609294341296.png)





#### 部署方式

##### ServiceMesh

![1609294394316](MicroserviceDistributedSystem.assets/1609294394316.png)



##### 集中式 Proxy/LB

![1609294459482](MicroserviceDistributedSystem.assets/1609294459482.png)



##### Ingress/Egress Proxy

![1609294490831](MicroserviceDistributedSystem.assets/1609294490831.png)



##### 混合式 Hybrid

![1609294526895](MicroserviceDistributedSystem.assets/1609294526895.png)



#### Envoy的性能 

##### 延迟百分位

- https://www.loggly.com/blog/benchmarking-5-popular-load-balancers-nginx-haproxy-envoy-traefik-and-alb/

![1609294642315](MicroserviceDistributedSystem.assets/1609294642315.png)



##### 并发吞吐性能

- https://www.loggly.com/blog/benchmarking-5-popular-load-balancers-nginx-haproxy-envoy-traefik-and-alb/

![1609294688399](MicroserviceDistributedSystem.assets/1609294688399.png)





### Envoy 在 Lyft 的实践

#### Lyft - 7 年前

![1609294752948](MicroserviceDistributedSystem.assets/1609294752948.png)



#### Lyft - 5 年前

![1609294800493](MicroserviceDistributedSystem.assets/1609294800493.png)



#### Lyft 微服务架构的问题（5年前）

- **缺乏一致的服务治理**
- 多种语言栈和框架
- 多种协议（HTTP/1，HTTP/2，gRPC，DB,Caching, etc.)
- 黑盒子负载均衡器(AwSELB)
- 缺乏一致的监控( stats, tracing, logging )
- 缺乏一致的容错限流（retry,circuit breaking,rate limiting, timeouts )
- 服务间几乎没有认证和授权机制
- 需要为不同的语言栈开发客户端
- 对延迟和失败的监控不完善
- 开发人员对微服务架构缺乏信心



#### 引入 Envoy + ServiceMesh

- 每一个服务使用一个 边车

![1609294946137](MicroserviceDistributedSystem.assets/1609294946137.png)



#### Envoy 核心价值

- 所有流量集中经过Envoy
  - 统一的监控metrics/logging/tracing
  - 统一传播request ID/tracing context
  - 集中的流量治理
  - 集中的限流熔断
  - 集中的安全治理
- 统一的服务治理是Envoy+ServiceMesh 提供的最主要价值



#### Lyft 2018

![1609295095390](MicroserviceDistributedSystem.assets/1609295095390.png)



#### 单个服务监控面板

![1609295134152](MicroserviceDistributedSystem.assets/1609295134152.png)



#### 分布式调用链

![1609295158799](MicroserviceDistributedSystem.assets/1609295158799.png)



#### 日志

![1609295176088](MicroserviceDistributedSystem.assets/1609295176088.png)



#### 服务到服务的监控

![1609295203264](MicroserviceDistributedSystem.assets/1609295203264.png)



#### 对 Envoy Proxy 的监控

![1609295228465](MicroserviceDistributedSystem.assets/1609295228465.png)



#### 全局健康监控

![1609295250447](MicroserviceDistributedSystem.assets/1609295250447.png)



#### 瘦客户端 @Lyft

![1609295313475](MicroserviceDistributedSystem.assets/1609295313475.png)



#### 控制平面和配置管理

- 引入 Envoy manager service

![1609295342336](MicroserviceDistributedSystem.assets/1609295342336.png)



#### 总结和参考

- Envoy + ServiceMesh
  - 标准化通讯层和服务治理
  - 标准化监控
  - 支持多语言栈/框架和协议，简化客户端
  - 控制平面可定制自研
  - 中大规模企业考虑
- Lyft's Envoy: Embracing a Service Mesh
  - https://www.youtube.com/watch?v=55yi4MMVBi4
  - https://www.slideshare.net/InfoQ/lyfts-envoy-embracing-a-service-mesh. 
- lntroducing envoy-based service mesh at Booking.com
  - https://www.youtube.com/watch?v=Pus2ytdEfrQ
  - https://www2.slideshare.net/IvanKruglov/ivan-kruglov-introducing-envoybased-service-mesh-at-bookingcom-version-7?qid=fd839056-940a-4f0f-b2bb-ab9472d480b1&v=&b=&from_search=2



### 解析 Istio

#### Istio v1.6 架构

- 流量治理
- 安全治理
- 监控测量
- https://istio.io/latest/docs/ops/deployment/architecture/
- 数据平面 + 控制平面

![1609295954961](MicroserviceDistributedSystem.assets/1609295954961.png)





#### 流量治理

- 实现金丝雀发布，A/B测试

![1609296036047](MicroserviceDistributedSystem.assets/1609296036047.png)



#### 新概念 - VirtualService  & DestinationRule

- 引入虚拟服务，client 不需要关注具体的服务，保证接口的一致性

![1609296076878](MicroserviceDistributedSystem.assets/1609296076878.png)



#### 安全架构

- https://istio.io/latest/docs/concepts/security/

![1609296181276](MicroserviceDistributedSystem.assets/1609296181276.png)



#### 可视化监控 - Kiali

- https://istio.io/latest/docs/tasks/observability/kiali/

![1609296299839](MicroserviceDistributedSystem.assets/1609296299839.png)



#### 理解 k8s Service

- 一个Node 中，可以包含多个不同服务的 pod
- 内部服务请求，会经过公共的 集群 服务IP Table 进行路由

![1609296327776](MicroserviceDistributedSystem.assets/1609296327776.png)

- 使用 Kube-proxy 之后，将服务路由 IP 的功能，下沉到每一个 Node 中

![1609296474357](MicroserviceDistributedSystem.assets/1609296474357.png)



#### 理解 k8s + Istio

- 集成 Istio 之后，会有一个公共的控制面板 Istio Control Plane
- 每一个路由的proxy ,下沉到每一个 pod 中，使用 Istio proxy 进行服务的路由

![1609296611466](MicroserviceDistributedSystem.assets/1609296611466.png)

- 此时的  Istio proxy 功能，就是缓存整个 k8s 服务的路由

![1609296689784](MicroserviceDistributedSystem.assets/1609296689784.png)

- 当pod的服务有请求，直接在本地被截获，转发到对应的目标服务上

![1609296760088](MicroserviceDistributedSystem.assets/1609296760088.png)

- 当外部请求需要访问服务的时候，经过 LB后，会有一个 Istio Gateway，
- 作为请求的网关，内部也会有一个 Istio proxy ，可以直接命中请求的对象

![1609296877529](MicroserviceDistributedSystem.assets/1609296877529.png)



#### 参考

- k8s 基本概念和应用
- https://www.bilibili.com/video/BV1Ja4y1x748



### K8s Ingress、lstio Gateway和 APl Gateway该如何选择?

#### K8s ClusterIP Service - Userspace Proxy Mode

![1609297218067](MicroserviceDistributedSystem.assets/1609297218067.png)



#### K8s ClusterIP Service - Iptables Mode

![1609297282745](MicroserviceDistributedSystem.assets/1609297282745.png)



#### K8s ClusterIP Service - IPVS Mode

![1609297310205](MicroserviceDistributedSystem.assets/1609297310205.png)



#### Istio Sidecar Proxy

![1609297383134](MicroserviceDistributedSystem.assets/1609297383134.png)



#### NodePort

![1609297488537](MicroserviceDistributedSystem.assets/1609297488537.png)



#### LoadBalancer

![1609297564796](MicroserviceDistributedSystem.assets/1609297564796.png)



#### K8s Ingress

- 反向代理，路由功能

![1609297615682](MicroserviceDistributedSystem.assets/1609297615682.png)



####  LB + NodePort + Ingress

![1609297670248](MicroserviceDistributedSystem.assets/1609297670248.png)



#### 流量路径

![1609297717917](MicroserviceDistributedSystem.assets/1609297717917.png)



#### K8s Ingress 作为 API 网关

![1609297775936](MicroserviceDistributedSystem.assets/1609297775936.png)



#### Istio Gateway 作为 API 网关

![1609297823239](MicroserviceDistributedSystem.assets/1609297823239.png)



#### K8s Ingress vs lstio Gateway vs APl Gateway

![1609297882601](MicroserviceDistributedSystem.assets/1609297882601.png)





#### 综合方案

- lstio Gateway 作为API 和业务的网关
- APl Gateway 作为 API 的网关

![1609297915261](MicroserviceDistributedSystem.assets/1609297915261.png)



#### 参考

- 如何为服务网格选择入口网关
  - https://zhaohuabing.com/post/2019-03-29-how-to-choose-ingress-for-service-mesh/
- Kubernetes 网络三部曲
  - pod 网络：https://blog.csdn.net/yang75108/article/details/101101384
  - Service 网络：https://blog.csdn.net/yang75108/article/details/101101384
  - NodePort vs LoadBalancer vs Ingress：https://blog.csdn.net/yang75108/article/details/101101384
  - https://www.bilibili.com/video/BV1Ft4y117Ch



### Spring Cloud、K8s和Istio 该如何集成？

#### 微服务公共关注点

![1609298177591](MicroserviceDistributedSystem.assets/1609298177591.png)



#### 服务化演进历史

![1609298212229](MicroserviceDistributedSystem.assets/1609298212229.png)



#### 比较

![1609298248669](MicroserviceDistributedSystem.assets/1609298248669.png)



#### 推荐选择

![1609298267924](MicroserviceDistributedSystem.assets/1609298267924.png)



#### 集成 K8s 的企业级微服务技术中台架构

![1609298349433](MicroserviceDistributedSystem.assets/1609298349433.png)



## 大型网站架构演进案例

### 拍拍贷案例：大型网站架构是如何演进的？

#### 从一个小网站开始

- 拍拍贷参考了国外的 LendingClub
- 主要模块：借入，借出，账户 等

![1609377730506](MicroserviceDistributedSystem.assets/1609377730506.png)



#### 2014年之前

- 技术栈简单
- 1 web + 1 DB
- 自由发布
- 用户就是测试人员

![1609377790823](MicroserviceDistributedSystem.assets/1609377790823.png)



#### 单体拆分

- 流量/业务/团队开始增长
- 一个单体工程，发布频率受限
- 单体和团队规模矛盾
- 牵一发而动全身
- 业务交付缓慢

![1609377855860](MicroserviceDistributedSystem.assets/1609377855860.png)



#### 先拆应用

- 根据不同功能，拆分出不同的模块
- 一般先对数据表，进行垂直拆分

![1609377917230](MicroserviceDistributedSystem.assets/1609377917230.png)



#### 2014 - 2015 年

![1609377942483](MicroserviceDistributedSystem.assets/1609377942483.png)





#### 多应用的问题

-  移动业务站 与 PC业务站 无法进行重用，业务逻辑不同

![1609378025331](MicroserviceDistributedSystem.assets/1609378025331.png)





#### 服务化 1.0

- 基于.NET WCF
- 开始了服务化初阶阶段
- 仅是给其它应用提供服务
- 逻辑同时存在于应用和服务中
- WSDL服务应用方式调用
- 应用逻辑和调用关系变得复杂

![1609378098979](MicroserviceDistributedSystem.assets/1609378098979.png)



#### 服务化 2.0

- PPD 自研服务注册中心
- 标准 REST 服务
- 初步服务治理
- 引入无线 API 网关（反向路由，容错限流）

![1609378392774](MicroserviceDistributedSystem.assets/1609378392774.png)



#### 服务监控演进

- 自研 LogServer 
- 使用 ELK
- 最终使用 Elk+ CAT
- cat：https://github.com/dianping/cat

![1609378506325](MicroserviceDistributedSystem.assets/1609378506325.png)



#### 2016年底

- 多应用 -> 服务化
- 服务治理 + 监控

![1609378571585](MicroserviceDistributedSystem.assets/1609378571585.png)





#### 2017中台化战略

- 运维基础设施层
- 技术中台
- 业务中台
- 前台业务应用层

![1609378673168](MicroserviceDistributedSystem.assets/1609378673168.png)



#### 4年的变化

![1609378701159](MicroserviceDistributedSystem.assets/1609378701159.png)





### 最小可用架构（Minimun Viable Architecture）

#### 三个公司的架构演进

- ebay
- 携程
- 拍拍贷

![1609378820107](MicroserviceDistributedSystem.assets/1609378820107.png)



#### 行业模式

- 几乎没有公司一开始就搞微服务
- 到达一定的规模，必然演进出微服务
- 但是大部分公司（99%）都到不了那个规模



#### S 曲线

- [The S-Curve: How Businesses ACTUALLY Grow](https://ittybiz.com/s-curve/)：https://ittybiz.com/s-curve/

![1609379016973](MicroserviceDistributedSystem.assets/1609379016973.png)



#### 想法阶段

![1609379118692](MicroserviceDistributedSystem.assets/1609379118692.png)



##### 原型架构(Prototype Architecture)

- 目标: 快速低成本探索业务域
  - 找到业务模式
  - 找到产品市场契合点(product market fit)
  - 获取种子用户
- 快速迭代
  - 所有东西都是原型
  - 原型都可被丢弃
- 基本不需要技术
  - 纸上的原型
  - 谷歌广告
  - Excel表格
  - wordpress博客
- 如果确实要搞点开发
  - 简单熟悉的技术
  - 简单拼凑即可

![1609379261160](MicroserviceDistributedSystem.assets/1609379261160.png)



#### 起步阶段

![1609379309109](MicroserviceDistributedSystem.assets/1609379309109.png)



##### 刚好够用的架构(Just Enough Architecture）

- 目标: 以最低成本满足用户需求
  - 让第一批种子用户满意
  - 获取更多用户
- 快速学习和改进
- 提升团队生产力
- 先不要考虑规模化!
- 简单熟悉的技术
  - 功能够且易于使用
  - 快速开发框架（PHP/Django等）
- 单体架构
  - 单个应用
  - 单个数据库
- 不要自建基础设施
  - 使用云服务/PaaS/Serverless



##### 可牺牲的架构(Sacrifical Archetecture)

- https://martinfowler.com/bliki/SacrificialArchitecture.html

![1609379492801](MicroserviceDistributedSystem.assets/1609379492801.png)



##### 单体架构的利弊

![1609379657019](MicroserviceDistributedSystem.assets/1609379657019.png)





##### 尽量不要造轮子

- 使用云服务
  - 比自己造轮子更快，更便宜，也更好用
  - 像Netflix/StitchFix/WeWork这些互联网公司都没有自建的数据中心
- 尽量使用开源
  - 开源通常比商业软件要好用
- 使用第三方SaaS服务
  - 监控/告警
  - 项目管理/缺陷跟踪
  - 计费支付/欺诈检测
  - 等等
- 聚焦你的核心竞争
  ·其它都用XAAS



##### 起步阶段技术架构总结

尽量组装，不要自研



##### 提前为规模化做准备

- 模块化原则
  - 在单体中使用“共享库”
  - 易于修改和模块化
- 按应用垂直拆分
- 详细记录日志和埋点
  - 便于理解用户行为
  - 便于问题定位和恢复
  -  ELK/CAT



##### 功能开关

- 配置中心
  - eBay/Facebook/Google/Netflix/阿里/携程，都演进出自己的产品
  - Ctrip Apollo
- 将功能交付和代码交付进行解耦
  - 快速开启或关闭
  - 在生产环境中快速校验功能
  - 构建健壮+灵活的系统
- 可针对特定用户群开启或关闭功能
  - A|B测试
  - 培养试验文化

![1609380011490](MicroserviceDistributedSystem.assets/1609380011490.png)



##### 持续交付CD

- 低风险、push-button快速发布和回滚
- 大部分应用可以每天发布多次
- Docker/K8s

![1609380081717](MicroserviceDistributedSystem.assets/1609380081717.png)



#### 规模化阶段

![1609380132734](MicroserviceDistributedSystem.assets/1609380132734.png)



##### 什么时候考虑解耦拆分?

- 速度信号
  - 由于单体系统的耦合性和缺乏隔离性，交付速度受到影响
  - 团队间相互依赖明显，无法独立开发和交付
  - 新入职工程师要花很长时间才能上手交付
- 扩展性信号
  - 对单体的简单垂直扩展已经不起作用
  - 业务要求对系统的某部分进行独立扩展
- 发布信号
  - 业务要求对单体系统的某部分进行独立的发布
  - 但是单体发布很慢，很复杂，风险很高



##### 可扩展的架构(Scalable Architecture)

- 目标:技术架构要提前于业务的成长
  - 同时保持站点稳定(keep the site up) !
- 团队规模化
- 技术规模化
- 流程规模化



##### 两个披萨团队

![1609380310435](MicroserviceDistributedSystem.assets/1609380310435.png)



##### 可扩展的架构

- 可扩展的技术
  - 转型到{Java, Go,Python }
  - 优化延迟和性能
- 可扩展的服务层
  - 商品/订单/支付等
  - 微服务架构
- 可扩展的持久层
  - 拆分单体数据库
  - 隔离持久化
- 专用系统
  - 大数据分析/Al，搜索引擎，等等
- 发布订阅事件系统
  - 应用和服务之间通过事件进行通讯
  - 事件驱动架构(EDA)



##### 微服务架构的利弊

![1609380425467](MicroserviceDistributedSystem.assets/1609380425467.png)



##### 重构心态

![1609380445764](MicroserviceDistributedSystem.assets/1609380445764.png)





#### 优化阶段

![1609380492699](MicroserviceDistributedSystem.assets/1609380492699.png)



##### 稳定的架构

- 目标：让一个系统更加高效 + 可持续
- 持续增量地改善功能
- 持续提升技术效能
- 整合团队



#### 加入时候的公司阶段

- 规模化阶段

![1609380610193](MicroserviceDistributedSystem.assets/1609380610193.png)



#### 当前的公司阶段

- 优化阶段

![1609380673595](MicroserviceDistributedSystem.assets/1609380673595.png)



#### 架构心态

![1609380710765](MicroserviceDistributedSystem.assets/1609380710765.png)



#### 参考

- 最小可用架构
  - https://www.youtube.com/watch?v=-GysBkOxQiA
  - https://www.slideshare.net/RandyShoup/minimal-viable-architecture-silicon-slopes-2020



### 如何构建基于 Oauth2/JWT的微服务架构

#### ACME 电商平台架构

![1609380931731](MicroserviceDistributedSystem.assets/1609380931731.png)



#### 微服务分层架构

##### Nginx 反向代理层

- Nginx Ingress
- envoy

![1609381059627](MicroserviceDistributedSystem.assets/1609381059627.png)



##### web 应用层

- node js
- spring mvc

![1609381100823](MicroserviceDistributedSystem.assets/1609381100823.png)



##### API 网关层

- zuul

![1609381148467](MicroserviceDistributedSystem.assets/1609381148467.png)



##### IDP 服务

- 独立部署，实现 OAuth 的鉴权

![1609381233502](MicroserviceDistributedSystem.assets/1609381233502.png)



##### BFF 服务

- backend for frontend

![1609381253707](MicroserviceDistributedSystem.assets/1609381253707.png)



##### 领域服务层

- 分别部署领域的服务，之间不相互依赖，每一个领域都作为一个独立发布的单元

![1609381318673](MicroserviceDistributedSystem.assets/1609381318673.png)



#### 集成场景

##### 第一方 Web 应用 + 资源拥有者凭据模式

- 认证授权流程

![1609381434703](MicroserviceDistributedSystem.assets/1609381434703.png)

- 服务调用流程

![1609381497411](MicroserviceDistributedSystem.assets/1609381497411.png)



##### 第一方移动应用 + 授权码许可模式

- 认证授权流程

![1609381546129](MicroserviceDistributedSystem.assets/1609381546129.png)



##### 第三方 Web 应用 + 授权码模式

- 认证授权流程

![1609381595881](MicroserviceDistributedSystem.assets/1609381595881.png)



##### 额外说明

- IDP要支持从OAuth2令牌到JWT 令牌的互转
- sso单点登录
  - IDP提供集中登录页
  - IDP要支持Web Session会话
  - 粘性/客户端/集中式会话
- IDP和网关的部署方式
  - IDP在网关后
  - IDP在网关前/Nginx后
- 刷新令牌
  - 根据企业安全需求启用
  - 设置合理过期时间



#### 参考

- The Oauth 2.0 Authorization Framework
  - https://tools.ietf.org/html/rfc6749
- Micro-Services Architecture with OAuth2 & JWT
  - https://www.kaper.com/cloud/micro-services-architecture-with-oauth2-and-jwt-part-1-overview/
  - https://www.kaper.com/cloud/micro-services-architecture-with-oauth2-and-jwt-part-2-ateway/
  - https://www.kaper.com/cloud/micro-services-architecture-with-oauth2-and-jwt-part-3-idp/
- 什么是PKCE
  - https://dzone.com/articles/what-is-pkce



### 拍拍贷案例：如何实现机房的迁移？

#### 拍拍贷机房迁移挑战（2017下半年）

- 尽量不停机，进行机房迁移

![1609382291613](MicroserviceDistributedSystem.assets/1609382291613.png)



#### 通过波分光纤实现跨机房局域网

- 波分光纤：https://carrier.huawei.com/cn/products/fixed-network/transmission/wdm-otn

![1609382320337](MicroserviceDistributedSystem.assets/1609382320337.png)



### 携程/Netflix 案例：如何实现同城双活和异地多活

#### 携程准同城双活

![1609382749127](MicroserviceDistributedSystem.assets/1609382749127.png)



#### Netflix 同城三活高可用 + 混乱大猩猩

- 混乱大猩猩，进行主动制造故障与混乱

![1609382799451](MicroserviceDistributedSystem.assets/1609382799451.png)



#### Netflix 异地多活高可用

- 跨越多个Regin

![1609382861771](MicroserviceDistributedSystem.assets/1609382861771.png)



#### Eureka 服务发现高可用

![1609382904278](MicroserviceDistributedSystem.assets/1609382904278.png)



#### 缓存 EVCache 高可用

- 使用 sidecar 边车 的方式部署到每一个 memcached 服务中

![1609382961040](MicroserviceDistributedSystem.assets/1609382961040.png)



#### Zuul 网关跨区域弹性路由

- Zuul 可以判断本地的服务是否可用
- 进行动态的切换路由访问，保证高可用

![1609383028636](MicroserviceDistributedSystem.assets/1609383028636.png)



#### 参考

- High Availability Architecture and Netflixoss
  - https://www.slideshare.net/adrianco/high-availability-architecture-and-netflixoss/38-Denominator_The_next_version_is
- Active-Active for Multi-Regional Resiliency
  - https://netflixtechblog.com/active-active-for-multi-regional-resiliency-c47719f6685b. 
- Netflix Shares Cloud Load Balancing And Failover Tool: Eureka!
  - https://netflixtechblog.com/netflix-shares-cloud-load-balancing-and-failover-tool-eureka-c10647ef95e5
- Netflix EVcache Wiki
  - https://github.com/Netflix/EVCache/wiki· 
- Zuul @ Netflix
  -  https://www.slideshare.net/MikeyCohen1/zuul-netflix-springone-platform





## 架构师成长之道

### 学习开源项目的6个层次和8种方法

![1609384043346](MicroserviceDistributedSystem.assets/1609384043346.png)



#### 方法1-简单看源码

| 层次        | 一                                                           |
| ----------- | ------------------------------------------------------------ |
| 方法        | 简单看源码                                                   |
| 说明        | 简单看源码、文档、跑跑样例代码                               |
| 学习效果    | 1～ 2                                                        |
| 有产出+闭环 | 否/尽量问题驱动                                              |
| Get技能     | 源码阅读技能                                                 |
| 案例        | 简单小项目，大部分新手开发者 举例，一致性Hash算法 <br />https://github.com/Jaskey/ConsistentHash |

![1609384089034](MicroserviceDistributedSystem.assets/1609384089034.png)



#### 方法2-整理源代码

| 层次        | 二                                                           |
| ----------- | ------------------------------------------------------------ |
| 方法        | 整理源代码                                                   |
| 说明        | 将所有源代码整理一遍，包括将包名都改掉，编译/测试/运行 通过  |
| 学习效果    | 2～ 3                                                        |
| 有产出+闭环 | 否(假定没有输出分享)                                         |
| Get技能     | 细粒度源码阅读+调试                                          |
| 案例        | 波波看源码习惯，案例spring-petclinic <br />https://github.com/spring-projects/spring-petclinic <br />https://github.com/spring2go/spring-petclinic-mono |



#### 方法3-整理+输出分享

| 层次        | 三                                                           |
| ----------- | ------------------------------------------------------------ |
| 方法        | 整理+输出分享                                                |
| 说明        | 源码整理过，并且总结成源码分析或者架构设计文档/ppt，在公司或者社区 分享。 |
| 学习效果    | 3~5                                                          |
| 有产出+闭环 | 有价值输出+反馈                                              |
| Get技能     | 技术/整理/输出/分享技能                                      |
| 案例        | 波波看源码习惯，案例 Spring Petclinic <br />https://github.com/spring-petclinic/spring-petclinic-microservices <br />https://github.com/spring2go/spring-petclinic-msa <br />https://www.bilibili.com/video/BV1o5411x7ML <br />PPD 新手工程师每月源码解析文章 <br />https://techblog.ppdai.com/ |



##### Youtube 案例1

![1609384302097](MicroserviceDistributedSystem.assets/1609384302097.png)



##### Youtube 案例2

![1609384328421](MicroserviceDistributedSystem.assets/1609384328421.png)





#### 方法4-开发克隆版

| 层次        | 三                                                           |
| ----------- | ------------------------------------------------------------ |
| 方法        | 开发克隆版                                                   |
| 说明        | 开发原项目的克隆版，类似翻译，比方说原项目用 Go 开发，我克隆成 Java 版，并开源分享到社区。 |
| 学习效果    | 5~10                                                         |
| 有产出+闭环 | 有价值输出+社区反馈                                          |
| Get技能     | 深度源码阅读 技术/项目开发技能 Find high-quality projects on github |
| 案例        | Staffjoy 项目 [Spring Boot 与 Kubernetes 云原生微服务实践] <br />https://github.com/Staffjoy/v2 https://github.com/spring2go/staffjoy <br />Kafka 克隆项目 <br />https://github.com/adyliu/jafka <br />https://github.com/travisjeffery/jocko <br />https://github.com/bulldog2011/luxun |



#### 方法5-生产化落地

| 层次        | 四                                                           |
| ----------- | ------------------------------------------------------------ |
| 方法        | 生产化落地                                                   |
| 说明        | 源码经过整理输出，并且在公司生产级项目中落地，承担业务流量，并且根 据业务需要进行定制或者自研。 |
| 学习效果    | >10                                                          |
| 有产出+闭环 | 有价值输出+用户反馈+持续改进                                 |
| Get技能     | 项目实战落地技能 企业级源码阅读和定制技能                    |
| 案例        | 携程 Zuul 网关[微服务架构实战160讲] <br />https://github.com/spring2go/s2g-zuul <br />携程 Hystrix 限流熔断[微服务架构实战160讲] <br />https://github.com/ctripcorp/chystrix.net <br />拍拍贷 PMQ 消息系统[新课程第3章案例] <br />https://github.com/ppdaicorp/pmq <br />饿了么监控应用系统 <br />https://qcon.infoq.cn/2019/shanghai/presentation/1902 <br />https://myslide.cn/slides/21832 <br />https://github.com/lindb/lindb |



#### 方法6-开发知识产品

| 层次        | 四                                                           |
| ----------- | ------------------------------------------------------------ |
| 方法        | 开发知识产品                                                 |
| 说明        | 将开源项目代码完全吸收，开发克隆项目/或开发开源应用演示项目，并制作 成知识付费产品（视频/书籍），发布到商业平台。 |
| 学习效果    | >10                                                          |
| 有产出+闭环 | 有产品输出+用户反馈+持续改进                                 |
| Get技能     | 技术和项目开发技能 知识产品设计和开发技能 行业影响力         |
| 案例        | Staffjoy 项目[Spring Boot 与 Kubernetes 云原生微服务实践] <br />https://github.com/spring2go/staffjoy <br />新峰商城 <br />https://github.com/newbee-ltd/newbee-mall <br />https://juejin.im/book/5da2f9d4f265da5b81794d48?referrer=59199e22a22b9d0058279886 <br />https://edu.csdn.net/course/play/26258/326466 |



#### 方法7-开发自己的开源项目

| 层次        | 五                                                           |
| ----------- | ------------------------------------------------------------ |
| 方法        | 开发自己的开源项目                                           |
| 说明        | 在吸收开源项目+企业落地实践的基础上，沉淀出自己的开源项目，并持 续推动开源社区建设. |
| 学习效果    | >10                                                          |
| 有产出+闭环 | 有产品输出+社区反馈+持续改进                                 |
| Get技能     | 技术和项目开发技能 行业影响力，跳槽最佳姿势 GitHub is your resume, let HR find you! 开源和社区运营技能 |
| 案例        | Apollo [微服务架构实战160讲] <br />https://github.com/ctripcorp/apollo <br />Skywalking <br />https://github.com/apache/skywalking <br />https://wu-sheng.github.io/me/ <br />LinDB <br />https://github.com/lindb/lindb |



#### 方法8-商业化自己的开源项目

| 层次        | 六                                                           |
| ----------- | ------------------------------------------------------------ |
| 方法        | 商业化自己的开源项目                                         |
| 说明        | 在自己的开源产品基础上，持续推进社区生态建设，并逐步走上商业化服务的道路。 |
| 学习效果    | >100                                                         |
| 有产出+闭环 | 完全和企业客户闭环                                           |
| Get技能     | 产品化和商业化技能/社区生态建设技能/技术领导力+++            |
| 案例        | TiDB（2018.9 C轮5000万美元） <br />https://github.com/pingcap/tidb<br />https://pingcap.com/en/ <br />Kafka（2019.1 D轮1.25亿美金，估值25亿美金） <br />https://github.com/apache/kafka <br />https://www.confluent.io/ <br />Sentry（2019.9 C轮4000万美金） <br />https://github.com/getsentry <br />https://sentry.io |

![1609384831166](MicroserviceDistributedSystem.assets/1609384831166.png)



#### 要点心得

1.输入要有输出，学习要有产出。

2.闭环反馈+持续改进，用户越多，反馈越多，学习效果越好。最好你的学习效果能够量化。

3.如果你要真正理解某人/事物，那么就尝试改变他/她/它，学习开源项目也一样。

4.一举多得，投入时间->有价值产出->个人沉淀积累->贡献社区->提升影响力->真正学到东西。



#### 复利曲线

- 学技术，学英语，做开源，做视频课程，都遵循这条复利曲线
- 技术一般需要15年的时间，到拐点
- 短视频，需要3-5年

![1609385021078](MicroserviceDistributedSystem.assets/1609385021078.png)



### 百万架构师是如何炼成的？

#### 1 - 2 家规模化阶段的公司

![1609385166647](MicroserviceDistributedSystem.assets/1609385166647.png)



#### 3 - 5 个核心技术或者业务系统

![1609385206552](MicroserviceDistributedSystem.assets/1609385206552.png)



#### 至少 2 个业务或者技术领域经验

- 经验可以做到互吃
- 至少一个业务 + 一个技术

![1609385266800](MicroserviceDistributedSystem.assets/1609385266800.png)



#### 谋事在天，成事在人

- 谋事在人，成事在天

![1609385307329](MicroserviceDistributedSystem.assets/1609385307329.png)





### 解读一份大厂的研发岗职级体系

#### P1 - P3 - 培养层

- 培养层

![1609385369329](MicroserviceDistributedSystem.assets/1609385369329.png)

![1609385389250](MicroserviceDistributedSystem.assets/1609385389250.png)



#### P4 - P5 - 主力实施层

- 主力实施层

![1609385443899](MicroserviceDistributedSystem.assets/1609385443899.png)

![1609385458047](MicroserviceDistributedSystem.assets/1609385458047.png)



#### P6 - P7 - 主力交付层

- 主力交付层

![1609385490012](MicroserviceDistributedSystem.assets/1609385490012.png)

![1609385503083](MicroserviceDistributedSystem.assets/1609385503083.png)



#### P8 - P9  - 核心管理/ 总监层

- 核心管理/ 总监层

![1609385550527](MicroserviceDistributedSystem.assets/1609385550527.png)

![1609385565302](MicroserviceDistributedSystem.assets/1609385565302.png)



#### P10 - P12 - CTO 技术战略层

- CTO 技术战略层

![1609385598818](MicroserviceDistributedSystem.assets/1609385598818.png)

![1609385612792](MicroserviceDistributedSystem.assets/1609385612792.png)



### 结束语

#### 回顾

![1609385689576](MicroserviceDistributedSystem.assets/1609385689576.png)



#### 分布式系统设计原则

- 高可用、高性能、可扩展
- 分布式系统第一原则～能不要分布尽量不要分布/最小可用架构
- 尽可能无状态，其次集中状态，其次分布式最终一致，最终分布式强一致
- 尽量异步
- 测量反馈驱动（治理)



#### 推荐书籍

- 深入理解计算机系统
- 代码大全
- UML 和模式应用
- 架构即未来
- 分布式数据库
  - 数据密集型应用系统设计
  - 数据库系统内幕

![1609385876956](MicroserviceDistributedSystem.assets/1609385876956.png)

![1609385886028](MicroserviceDistributedSystem.assets/1609385886028.png)









