# ETCD 介绍
etcd 是开源的可信赖、高可用分布式键值存储系统，为分布式系统存储关键数据，协调整个分布式集群正常运转
## Raft 协议介绍
Raft 协议是分布式一致性协议，在协议实现过程中，通常包括三个角色
`Leader`: leader 通常执行分布式系统中的写入动作，然后通过日志复制的形式，同步到follower节点
`Follower`: follower 通常执行读取动作
'Candidate': raft 协议中选择leader 时的候选人
## ETCD API 语义
- Put(key,value)/Delete(key)
- Get(key)/Get(keyFrom,keyEnd)
- Watch(key/keyPrefix)
- Transactions(if/then/else ops).commit()
- Lease: Grant/Revoke/keepAlive

## ETCD 应用场景
- 分布式锁
- 服务注册与发现
- 消息发布与订阅
- 分布式通知与协调
  集群系统选主的场景，各应用可以通过抢写自己信息的形式，实现抢主。
### 租约的场景
lease 是分布式系统中一个常见的概念，用于代表一个分布式租约。典型情况下，在分布式系统中需要去检测一个节点是否存活的时，就需要租约机制。
![image](https://user-images.githubusercontent.com/5925204/139103957-a64ae379-b3f6-4dd8-8c95-2e7b7fecab5d.png)

上图示例中的代码示例首先创建了一个 10s 的租约，如果创建租约后不做任何的操作，那么 10s 之后，这个租约就会自动过期。接着将 key1 和 key2 两个 key value 绑定到这个租约之上，这样当租约过期时 etcd 就会自动清理掉 key1 和 key2，使得节点 key1 和 key2 具备了超时自动删除的能力。

如果希望这个租约永不过期，需要周期性的调用 KeeyAlive 方法刷新租约。比如说需要检测分布式系统中一个进程是否存活，可以在进程中去创建一个租约，并在该进程中周期性的调用 KeepAlive 的方法。如果一切正常，该节点的租约会一致保持，如果这个进程挂掉了，最终这个租约就会自动过期。

在 etcd 中，允许将多个 key 关联在同一个 lease 之上，这个设计是非常巧妙的，可以大幅减少 lease 对象刷新带来的开销。试想一下，如果有大量的 key 都需要支持类似的租约机制，每一个 key 都需要独立的去刷新租约，这会给  etcd 带来非常大的压力。通过多个 key 绑定在同一个 lease 的模式，我们可以将超时间相似的 key 聚合在一起，从而大幅减小租约刷新的开销，在不失灵活性同时能够大幅提高 etcd 支持的使用规模。
## 引用
[etcd-introduction](https://draveness.me/etcd-introduction/)
