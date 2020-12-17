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









































































































































































































