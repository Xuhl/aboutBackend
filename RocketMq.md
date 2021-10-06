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
