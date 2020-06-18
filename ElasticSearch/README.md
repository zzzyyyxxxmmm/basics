# What is Elastic Search
Elastics Search 是实时的分布式搜索引擎, 内部使用Lucene做索引与搜索. Lucene是Java语言比那些的全文搜索框架, 用于处理纯文本的数据, 但他只是一个库, 提供建立索引, 执行搜索等接口, 但不包含分布式服务.

Elasticsearch is the distributed search and analytics engine and provides real-time search and analytics for all types of data. 

While not every problem is a search problem, Elasticsearch offers speed and flexibility to handle data in a wide variety of use cases:

* Add a search box to an app or website
* Store and analyze logs, metrics, and security event data
* Use machine learning to automatically model the behavior of your data in real time
* Automate business workflows using Elasticsearch as a storage engine
* Manage, integrate, and analyze spatial information using Elasticsearch as a geographic information system (GIS)
* Store and process genetic data using Elasticsearch as a bioinformatics research tool

Elasticsearch is a distributed document store. Instead of storing information as rows of columnar data, Elasticsearch stores complex data structures that have been serialized as JSON documents. When you have multiple Elasticsearch nodes in a cluster, stored documents are distributed across the cluster and can be accessed immediately from any node.

When a document is stored, it is indexed and fully searchable in near real-time—​within 1 second. Elasticsearch uses a data structure called an inverted index that supports very fast full-text searches. An inverted index lists every unique word that appears in any document and identifies all of the documents each word occurs in.

# 集群启动流程
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/es_1.png" width="700" height="500">
</div>

## 选举主节点
ES的选主算法是基于Bully算法的改进, 主要思路是对节点ID排序, 取ID值最大的节点作为Master.

和raft不同的是, 即使出现网络分区, 集群仍然只有一个master节点, 因此要求, 当探测到节点离开事件时, 必须判断当前节点数是否过半. 如果达不到quorum, 则放弃Master身份, 重新加入集群.

## 选举集群元信息
被选出的Master和集群元信息的新旧程度没有关系, 因此它的第一个任务是选举元信息, 让各节点把格子存储的元信息发过来, 根据版本号确定最新的元信息, 然后把这个信息广播下去, 这样集群的所有节点都有了最新的元信息. 

集群元信息的选举包括两个级别: 集群级和索引级. 不包含哪个shard存于哪个节点这种信息. 集群元信息选举完毕后, Master发布首次集群状态, 然后开始选举shard级元信息

## allocation过程
选举shard级元信息，构建内容路由表，是在allocation模块完成的。在初始阶段，所有的shard都处于UNASSIGNED（未分配）状态。ES中通过分配过程决定哪个分片位于哪个节点，重构内容路由表。此时，首先要做的是分配主分片。主分片选举过程是通过集群级元信息中记录的“最新主分片的列表”来确定主分片的.

主分片选举完成后，从上一个过程汇总的 shard 信息中选择一个副本作为副分片。如果汇总信息中不存在，则分配一个全新副本的操作依赖于延迟配置项：

index.unassigned.node_left.delayed_timeout

我们的线上环境中最大的集群有100+节点，掉节点的情况并不罕见，很多时候不能第一时间处理，这个延迟我们一般配置为以天为单位。

## index recovery
分片分配成功后进入recovery流程。主分片的recovery不会等待其副分片分配成功才开始recovery。它们是独立的流程，只是副分片的recovery需要主分片恢复完毕才开始。

为什么需要recovery？对于主分片来说，可能有一些数据没来得及刷盘；对于副分片来说，一是没刷盘，二是主分片写完了，副分片还没来得及写，主副分片数据不一致。

1. 主分片recovery

由于每次写操作都会记录事务日志（translog），事务日志中记录了哪种操作，以及相关的数据。因此将最后一次提交（Lucene 的一次提交就是一次 fsync 刷盘的过程）之后的 translog中进行重放，建立Lucene索引，如此完成主分片的recovery。

2. 副分片recovery

副分片的恢复是比较复杂的，在ES的版本迭代中，副分片恢复策略有过不少调整。

副分片需要恢复成与主分片一致，同时，恢复期间允许新的索引操作。在目前的6.0版本中，恢复分成两阶段执行。

* phase1：在主分片所在节点，获取translog保留锁，从获取保留锁开始，会保留translog不受其刷盘清空的影响。然后调用Lucene接口把shard做快照，这是已经刷磁盘中的分片数据。把这些shard数据复制到副本节点。在phase1完毕前，会向副分片节点发送告知对方启动engine，在phase2开始之前，副分片就可以正常处理写请求了。

* phase2：对translog做快照，这个快照里包含从phase1开始，到执行translog快照期间的新增索引。将这些translog发送到副分片所在节点进行重放。

由于需要支持恢复期间的新增写操作（让ES的可用性更强），这两个阶段中需要重点关注以下几个问题。

# 节点关闭流程
现在我们探讨一下单个节点的关闭流程。设想当我们为 ES 集群更新配置、升级版本时，需要通过“kill”ES进程来关闭节点。但是kill操作是否安全？如果此时节点有正在执行的读写操作会有什么影响？如果节点是Master该如何处理？关闭流程是怎么实现的？kill节点都会带来哪些风险？

答案是：ES进程会捕获SIGTERM信号（kill命令默认信号）进行处理，调用各模块的stop方法，让它们有机会停止服务，安全退出。

关闭顺序大致如下：

* 关闭快照和HTTPServer，不再响应用户REST请求。
* 关闭集群拓扑管理，不再响应ping请求。
* 关闭网络模块，让节点离线。
* 执行各个插件的关闭流程。
* 关闭IndicesService。

最后才关闭IndicesService，是因为这期间需要等待释放的资源最多，时间最长。 关闭快照和HTTPServer，不再响应用户REST请求。

最后才关闭IndicesService，是因为这期间需要等待释放的资源最多，时间最长。

## 分片读写过程中执行关闭
下面分别对读和写执行过程中关闭节点进行分析。

写入过程中关闭：线程在写入数据时，会对Engine加写锁。IndicesService的doStop方法对本节点上全部索引并行执行removeIndex，当执行到Engine的flushAndClose（先flush然后关闭Engine），也会对Engine加写锁。由于写入操作已经加了写锁，此时写锁会等待，直到写入执行完毕。因此数据写入过程不会被中断。但是由于网络模块被关闭，客户端的连接会被断开。客户端应当作为失败处理，虽然ES服务端的写流程还在继续。可能来不及通知客户端

读取过程中关闭：线程在读取数据时，会对Engine加读锁。flushAndClose时的写锁会等待读取过程执行完毕。但是由于连接被关闭，无法发送给客户端，导致客户端读失败。

# 选主流程

## 流程概述
ZenDiscovery的选主过程如下：

* 每个节点计算最小的已知节点ID，该节点为临时Master。向该节点发送领导投票。

* 如果一个节点收到足够多的票数，并且该节点也为自己投票，那么它将扮演领导者的角色，开始发布集群状态。

所有节点都会参与选举，并参与投票，但是，只有有资格成为Master的节点（node.master为true）的投票才有效．

获得多少选票可以赢得选举胜利，就是所谓的法定人数。在 ES 中，法定大小是一个可配置的参数。配置项：discovery.zen.minimum_master_nodes。为了避免脑裂，最小值应该是有Master资格的节点数n/2+1。

## 流程分析
每个节点选出自己认为的临时master(基于ID), 发送加入请求(即投票), 超过一半则当选

到此为止，选主流程已执行完毕，Master身份已确认，非Master节点已加入集群。

节点失效检测会监控节点是否离线，然后处理其中的异常。失效检测是选主流程之后不可或缺的步骤，不执行失效检测可能会产生脑裂（双主或多主）。在此我们需要启动两种失效探测器：

* 在Master节点，启动NodesFaultDetection，简称NodesFD。定期探测加入集群的节点是否活跃。主要判断剩余节点是否达到法定人数

* 在非Master节点启动MasterFaultDetection，简称MasterFD。定期探测Master节点是否活跃。离线则重新选举

NodesFaultDetection和MasterFaultDetection都是通过定期（默认为1秒）发送的ping请求探测节点是否正常的，当失败达到一定次数（默认为3次），或者收到来自底层连接模块的节点离线通知时，开始处理节点离开事件。

