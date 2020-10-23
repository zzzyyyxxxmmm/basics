# What is Zookeeper
Zookeeper是一个高性能分布式应用协调服务, 实现分布式锁

ZooKeeper was designed to be a robust service that enables application developers to focus mainly on their application logic rather than coordination. It exposes a simple API, inspired by the filesystem API, that allows developers to implement common coordination tasks, such as electing a master server, managing group membership, and managing metadata. 

**Apache HBase**

HBase is a data store typically used alongside Hadoop. In HBase, ZooKeeper is used to elect a cluster master, to keep track of available servers, and to keep cluster metadata.

**Apache Kafka**
Kafka is a pub-sub messaging system. It uses ZooKeeper to detect crashes, to im‐ plement topic discovery, and to maintain production and consumption state for topics.

**Apache Solr**
Solr is an enterprise search platform. In its distributed form, called SolrCloud, it uses ZooKeeper to store metadata about the cluster and coordinate the updates to this metadata.

**Yahoo! Fetching Service**
Part of a crawler implementation, the Fetching Service fetches web pages efficiently by caching content while making sure that web server policies, such as those in robots.txt files, are preserved. This service uses ZooKeeper for tasks such as master election, crash detection, and metadata storage.

**Facebook Messages**
This is a Facebook application that integrates communication channels: email, SMS, Facebook Chat, and the existing Facebook Inbox. It uses ZooKeeper as a controller for implementing sharding and failover, and also for service discovery.

# Zookeeper basic

We consequently have taken a different path with ZooKeeper. ZooKeeper does not ex‐ pose primitives directly. Instead, it exposes a file system-like API comprised of a small set of calls that enables applications to implement their own primitives. We typically use recipes to denote these implementations of primitives. Recipes include ZooKeeper operations that manipulate small data nodes, called znodes, that are organized hier‐ archically as a tree, just like in a file system. Figure 2-1 illustrates a znode tree. The root node contains four more nodes, and three of those nodes have nodes under them. The leaf nodes are the data.

## Different Modes for Znodes
When creating a new znode, you also need to specify a mode. The different modes determine how the znode behaves.
**Persistent and ephemeral znodes**

A znode can be either persistent or ephemeral. A persistent znode /path can be deleted only through a call to delete. An ephemeral znode, in contrast, is deleted if the client that created it crashes or simply closes its connection to ZooKeeper.
Persistent znodes are useful when the znode stores some data on behalf of an application and this data needs to be preserved even after its creator is no longer part of the system. For example, in the master-worker example, we need to maintain the assignment of tasks to workers even when the master that performed the assignment crashes.

Ephemeral znodes convey information about some aspect of the application that must exist only while the session of its creator is valid. For example, the master znode in our master-worker example is ephemeral. Its presence implies that there is a master and the master is up and running. If the master znode remains while the master is gone, then the system won’t be able to detect the master crash. This would prevent the system from making progress, so the znode must go with the master. We also use ephemeral znodes for workers. If a worker becomes unavailable, its session expires and its znode in /workers disappears automatically.
An ephemeral znode can be deleted in two situations:
1. When the session of the client creator ends, either by expiration or because it ex‐ plicitly closed.
2. When a client, not necessarily the creator, deletes it.

Because ephemeral znodes are deleted when their creator’s session expires, we currently do not allow ephemerals to have children. There have been discussions in the commu‐ nity about allowing children for ephemeral znodes by making them also ephemeral. This feature might be available in future releases, but it isn’t available currently.

**Sequential znodes**

A znode can also be set to be sequential. A sequential znode is assigned a unique, mo‐ notonically increasing integer. This sequence number is appended to the path used to create the znode. For example, if a client creates a sequential znode with the path /tasks/ task-, ZooKeeper assigns a sequence number, say 1, and appends it to the path. The path of the znode becomes /tasks/task-1. Sequential znodes provide an easy way to create znodes with unique names. They also provide a way to easily see the creation order of znodes.
To summarize, there are four options for the mode of a znode: persistent, ephemeral, persistent_sequential, and ephemeral_sequential.

## Watches and Notifications
Because ZooKeeper is typically accessed as a remote service, accessing a znode every time a client needs to know its content would be very expensive: it would induce higher latency and more operations to a ZooKeeper installation. Consider the example in Figure 2-2. The second call to getChildren /tasks returns the same value, an empty set, and consequently is unnecessary.

This is a common problem with polling. To replace the client polling, we have opted for a mechanism based on notifications: clients register with ZooKeeper to receive notifi‐ cations of changes to znodes. Registering to receive a notification for a given znode consists of setting a watch. A watch is a one-shot operation, which means that it triggers one notification. To receive multiple notifications over time, the client must set a new watch upon receiving each notification.

每次创建watcher的时候会重新读一遍value, 防止value过期

zookeeper也有version


在ZNode改变的时候, watcher会给客户端发送通知

## ZooKeeper Architecture
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/zk_1.png" width="700" height="500">
</div>

ZooKeeper Quorums

In quorum mode, ZooKeeper replicates its data tree across all servers in the ensemble. But if a client had to wait for every server to store its data before continuing, the delays might be unacceptable. In public administration, a quorum is the minimum number of legislators required to be present for a vote. In ZooKeeper, it is the minimum number of servers that have to be running and available in order for ZooKeeper to work. This number is also the minimum number of servers that have to store a client’s data before telling the client it is safely stored. For instance, we might have five ZooKeeper servers in total, but a quorum of three. So long as any three servers have stored the data, the client can continue, and the other two servers will eventually catch up and store the data.

### Session
Sessions offer order guarantees

One important parameter you should set when creating a session is the session timeout, which is the amount of time the ZooKeeper service allows a session before declaring it expired. If the service does not see messages associated to a given session during time t, it declares the session expired. On the client side, if it has heard nothing from the server at 1/3 of t, it sends a heartbeat message to the server. At 2/3 of t, the ZooKeeper client starts looking for a different server, and it has another 1/3 of t to find one.

When trying to connect to a different server, it is important for the ZooKeeper state of this server to be at least as fresh as the last ZooKeeper state the client has observed. A client cannot connect to a server that has not seen an update that the client might have seen. ZooKeeper determines freshness by ordering updates in the service. Every change to the state of a ZooKeeper deployment is totally ordered with respect to all other exe‐ cuted updates. Consequently, if a client has observed an update in position i, it cannot connect to a server that has only seen iʹ < i. In the ZooKeeper implementation, the transaction identifiers that the system assigns to each update establish the order.

Figure 2-7 illustrates the use of transaction identifiers (zxids) for reconnecting. After the client disconnects from s1 because it times out, it tries s2, but s2 has lagged behind and does not reflect a change known to the client. However, s3 has seen the same changes as the client, so it is safe to connect.

### Implementing a Primitive: Locks with ZooKeeper
One simple example of what we can do with ZooKeeper is implement critical sections through locks. There are multiple flavors of locks (e.g., read/write locks, global locks) and several ways to implement locks with ZooKeeper. Here we discuss a simple recipe just to illustrate how applications can use ZooKeeper; we do not consider other variants of locks.

Say that we have an application with n processes trying to acquire a lock. Recall that ZooKeeper does not expose primitives directly, so we need to use the ZooKeeper inter‐ face to manipulate znodes and implement the lock. To acquire a lock, each process p tries to create a znode, say /lock. If p succeeds in creating the znode, it has the lock and can proceed to execute its critical section. One potential problem is that p could crash and never release the lock. In this case, no other process will ever be able to acquire the lock again, and the system could seize up in a deadlock. To avoid such situations, we just have to make the /lock znode ephemeral when we create it.

Other processes that try to create /lock fail so long as the znode exists. So, they watch for changes to /lock and try to acquire the lock again once they detect that /lock has been deleted. Upon receiving a notification that /lock has been deleted, if a process pʹ is still interested in acquiring the lock, it repeats the steps of attempting to create /lock and, if another process has created the znode already, watching it.

### Implementation of a Master-Worker Example
1. server通过创建一个/master的enphermal node来表示当前是master, 其他尝试的server会得到该znode已被注册, 同时可以创建watch
2. create /workers /tasks /assign The three new znodes are persistent znodes and contain no data. We use these znodes in this example to tell us which workers are available, tell us when there are tasks to assign, and make assignments to workers. Regardless of how they are created, once they exist, the master needs to watch for changes in the children of /workers and /tasks:
3. worker在/worker创建, 表示该work就绪.
4. worker在/assign/worker1创建watcher, 观察等待分配到这个目录的任务
5. client创建/tasks/task-0000000000, 并watch
6. master watch/tasks, 并分配/assign/worker1.example.com/task-0000000000, worker观察到, 并处理, 然后创建/tasks/task-0000000000/status "done"
7. client观察到done, 表示处理成功

# Zookeeper internal
Client requests that change the state of ZooKeeper (create, delete, and setData) are forwarded to the leader.

Let’s now look at a ZooKeeper example. Say that a client submits a setData request on a given znode /z. setData should change the data of the znode and bump up the version number. So, a transaction for this request contains two important fields: the new data of the znode and the new version number of the znode. When applying the transaction, a server simply replaces the data of /z with the data in the transaction and the version number with the value in the transaction, rather than bumping it up.

A transaction is treated as a unit, in the sense that all changes it contains must be applied atomically. In the setData example, changing the data without an accompanying change to the version accordingly leads to trouble. Consequently, when a ZooKeeper ensemble applies transactions, it makes sure that all changes are applied atomically and there is no interference from other transactions. There is no rollback mechanism like with tra‐ ditional relational databases. Instead, ZooKeeper makes sure that the steps of transac‐ tions do not interfere with each other. For a long time, the design used a single thread in each server to apply transactions. Having a single thread guarantees that the trans‐ actions are applied sequentially without interference. Recently, ZooKeeper has added support for multiple threads to speed up the process of applying transactions.

A transaction is also idempotent. That is, we can apply the same transaction twice and we will get the same result. We can even apply multiple transactions multiple times and get the same result, as long as we apply them in the same order every time. We take advantage of this idempotent property during recovery.

When the leader generates a new transaction, it assigns to the transaction an identifier that we call a ZooKeeper transaction ID (zxid). Zxids identify transactions so that they are applied to the state of servers in the order established by the leader. Servers also exchange zxids when electing a new leader, so they can determine which nonfaulty server has received more transactions and can synchronize their states.
# Zookeeper的工作方式
* Zookeeper集群包含一个Leader, 多个Follower
* 所有的Follower都可提供读服务
* 所有的写操作都会被forward到Leader
* Client与Server通过NIO通信
* 全局串行化所有的写操作
* 保证同一客户端指令被FIFO执行
* 保证消息通知的FIFO

# Zab协议
## 广播模式
* Leader将所有更新(称为proposal), 顺序发送给Follower
* 当Leader收到半数以上的Follower对此proposal的ACK时, 即向所有Follower发送commit消息, 并在本地commit该消息
* Follower收到Proposal后即将该Proposal写入磁盘, 写入成功即返回ACK给Leader
* 每个Proposal都有一个唯一的单调递增的ProposalID, 即zxid

## 恢复模式
* 进入恢复模式: 当Leader宕机或者丢失大多数Follower后, 即进入恢复模式
* 结束恢复模式: 新领导被选举出来, 且大多数Follower完成了与Leader的状态同步后, 恢复模式立即结束, 从而进入广播模式
* **进入恢复模式的意义**
* 发现集群中被commit的proposal的最大zxid
* 建立新的epoch, 从而保证之前的Leader不能再commit新的Proposal
* 集群中大部分节点都被commit过前一个Leader commit过的消息, 而新的Leader是被大部分节点所支持的, 所以被之前Leader commit过的Proposal不会叠墅, 至少被一个节点所保存
* 新Leader会与所有的Follower通信, 从而保证大部分节点都拥有最新的数据