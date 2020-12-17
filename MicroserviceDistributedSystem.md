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



#### 数据库选型

- 可扩展
- 高性能
- 高可用
- 一致性
- 成本
- 学习门槛





#### SQL数据库+客户端嵌入代理

- ShardingSphere
- 将计数服务，路由到MySQL主节点上，写入数据
- 将查询服务，路由到MySQL从结点上，读取数据





#### SQL数据库+独立部署代理层

- ShardingSphere
- 记录数据库存储配置，有一个配置注册中心
- 分别进行计数服务和查询服务的路由





#### NoSQL数据库（Cassandra）

- 随机选择一个Node进行判断，写入的Node
- 仲裁写，多数写好，就表示已经写好了
- 一致性读，多个数据中心复制备份





#### 表设计

- SQL数据库，表之间的关联，join
- NOSQL数据库，数据冗余，





### 计数服务设计

#### 计数服务如何实现

- 可扩展性
- 高性能
- 高可用



#### 数据聚合（aggregation）基础

- 一般采用预聚合
- 请求pull，拉模式，引入消息队列



#### 消息队列基础

- Kafka
- 类似于数组，偏移量，消费指针，检查点
- 分区，对同一个主题，开多个分区，进行消息分摊



#### 计数消费者（详细设计）

- 数据聚合运算，使用内存中的concurrentHashmap，进行并发的运算
- 引入Internal的消息队列，作为数据库的缓冲
- DB writer 暂时无法写入数据到 DB 的时候，需要将数据写入 死信队列，保证数据不丢失
- 引入 Enrich Data Cache，将数据进行缓存，字段都写上值之后，在写入到数据库



#### 数据接收路径（Data Ingestion Path）

- API Gateway
- Counting Service，服务代理
- Kafka





#### 数据接收路径上的面试题

- API Gateway，软件Nginx，硬件F5，NDS，注册中心
- 容错限流，Histrix，TopK实时防爬虫
- 消息队列的格式，json，二进制





### 查询服务设计

#### 数据获取路径（Data Retrieval Path）

- 查询服务 Query Services
- 老数据归档（对象存储），2/8原则，近期数据缓存
- 热数据，冷数据



### 技术栈选型

#### 总体流程

- Counting service，计数服务
- Query service，查询服务



#### 技术栈选型

- API Gateway：f5，zuul
- Spring Boot
- Kafka
- redis
- cassandra，hadoop



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





#### 扩展

- 监控系统
- 欺诈检测系统
- 限流系统
- 推荐系统
- 今日热点



### 参考

- 
- Consistent Hash Implementation in Java：https://github.com/Jaskey/ConsistentHash





















































































































