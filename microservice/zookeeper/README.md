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

在ZNode改变的时候, watcher会给客户端发送通知
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