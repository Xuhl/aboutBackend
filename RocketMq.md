# RocketMq
## Namesrv 分析
Namesrv 核心入口在`NamesrvController`，
2个线程池
- 连接线程池
- 检测broker 失活
整个server 实现KV 存储，通过`KVConfigManager` 完成具体键值的存储。Namesrv 存储了broker 及topic 的路由信息，没有consumer 的信息
`RouteInfoManager` 管理最核心的数据，包括以下数据
- `HashMap<String, List<QueueData>> topicQueueTable` 存储topic 的路由信息
- ` HashMap<String, BrokerData> brokerAddrTable` 保存broker 路由信息
- ` HashMap<String/* brokerName */, BrokerData> brokerAddrTable` 集群信息
- `HashMap<String/* brokerAddr */, BrokerLiveInfo> brokerLiveTable` 活跃的broker 信息

## Broker 分析
Broker 有三种角色`org.apache.rocketmq.store.config.BrokerRole` : **ASYNC_MASTER**,**SYNC_MASTER**,**SLAVE**。SYNC_MASTER与ASYNC_MASTER的区别是sync会等待消息传输到slave才算消息写完成，而async不会同步等待，而是异步复制到slave

- ASYNC_MASTER、SLAVE：容许丢消息，但是要broker一直可用，master异步传输CommitLog到slave
- SYNC_MASTER、SLAVE：不允许丢消息，master同步传输CommitLog到slave
- ASYNC_MASTER：如果只是想简单部署则使用这种方式

Broker 启动的时候，会向NameSrv 注册信息（`org.apache.rocketmq.broker.BrokerController#doRegisterBrokerAll` 信息）

### 响应Producer 的sendMessage消息


SendMessage 在`SendMessageProcessor` 中处理,具体处理方法是`org.apache.rocketmq.broker.processor.SendMessageProcessor#processRequest`,


## 参考地址

https://xie.infoq.cn/article/9cfff95f682ddeb408a2b3928

https://www.itmuch.com/books/rocketmq/architecture.html

