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

# 数据模型

## PacificA算法

分片副本使用主从模式。多个副本中存在一个主副本Primary和多个从副本Secondary。所有的数据写入操作都进入主副本，当主副本出现故障无法访问时，系统从其他从副本中选择合适的副本作为新的主副本。

数据写入的流程如下：

1. 求进入主副本节点，节点为该操作分配SN，使用该SN创建UpdateRequest结构。然后将该UpdateRequest插入自己的prepare list。
2. 主副本节点将携带 SN 的 UpdateRequest 发往从副本节点，从节点收到后同样插入prepare list，完成后给主副本节点回复一个ACK。
3. 一旦主副本节点收到所有从副本节点的响应，确定该数据已经被正确写入所有的从副本节点，此时认为可以提交了，将此UpdateRequest放入committed list,committed list向前移动。
4. 主副本节点回复客户端更新成功完成。对每一个Prepare消息，主副本节点向从副本节点发送一个commit通知，告诉它们自己的committed point位置，从副本节点收到通知后根据指示移动committed point到相同的位置。

和Raft有点像

## ES的数据副本模型
ES 中的每个索引都会被拆分为多个分片，并且每个分片都有多个副本。这些副本称为replication group.

### 基本写入模型
写入过程遵循以下基本流程：

1. 请求到达协调节点，协调节点先验证操作，如果有错就拒绝该操作。然后根据当前集群状态，请求被路由到主分片所在节点。

2. 该操作在主分片上本地执行，例如，索引、更新或删除文档。这也会验证字段的内容，如果未通过就拒绝操作（例如，字段串的长度超出Lucene定义的长度）。

3. 操作成功执行后，转发该操作到当前in-sync 副本组的所有副分片。如果有多个副分片，则会并行转发。

4. 一旦所有的副分片成功执行操作并回复主分片，主分片会把请求执行成功的信息返回给协调节点，协调节点返回给客户端。

### 基本读取模型
基本流程如下：

（1）把读请求转发到相关分片。注意，因为大多数搜索都会发送到一个或多个索引，通常需要从多个分片中读取，每个分片都保存这些数据的一部分。

（2）从副本组中选择一个相关分片的活跃副本。它可以是主分片或副分片。默认情况下， ES会简单地循环遍历这些分片。

（3）发送分片级的读请求到被选中的副本。

（4）合并结果并给客户端返回响应。注意，针对通过ID查找的get请求，会跳过这个步骤，因为只有一个相关的分片。

# 写流程

## Index Bulk基本流程
以下是写单个文档所需的步骤：

（1）客户端向NODE1发送写请求。

（2）NODE1使用文档ID来确定文档属于分片0，通过集群状态中的内容路由表信息获知分片0的主分片位于NODE3，因此请求被转发到NODE3上。

（3）NODE3上的主分片执行写操作。如果写入成功，则它将请求并行转发到 NODE1和NODE2的副分片上，等待返回结果。当所有的副分片都报告成功，NODE3将向协调节点报告成功，协调节点再向客户端报告成功。

在客户端收到成功响应时，意味着写操作已经在主分片和所有副分片都执行完成。

写一致性的默认策略是quorum，即多数的分片（其中分片副本可以是主分片或副分片）在写入操作时处于可用状态。

## 详细流程
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/es_02.png" width="700" height="500">
</div>

# GET流程

## 可选参数
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/es_3.png" width="700" height="500">
</div>

## GET基本流程
（1）客户端向NODE1发送读请求。

（2）NODE1使用文档ID来确定文档属于分片0，通过集群状态中的内容路由表信息获知分片0有三个副本数据，位于所有的三个节点中，此时它可以将请求发送到任意节点，这里它将请求转发到NODE2。

（3）NODE2将文档返回给 NODE1,NODE1将文档返回给客户端。

NODE1作为协调节点，会将客户端请求轮询发送到集群的所有副本来实现负载均衡。 在读取时，文档可能已经存在于主分片上，但还没有复制到副分片。在这种情况下，读请求命中副分片时可能会报告文档不存在，但是命中主分片可能成功返回文档。一旦写请求成功返回给客户端，则意味着文档在主分片和副分片都是可用的。

**读失败是怎么处理的？** 尝试从别的分片副本读取。

## 详细分析
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/es_4.png" width="700" height="500">
</div>

## MGET流程
（1）遍历请求，计算出每个doc的路由信息，得到由shardid为key组成的request map。这个过程没有在TransportSingleShardAction中实现，是因为如果在那里实现，shardid就会重复，这也是合并为基于分片的请求的过程。

（2）循环处理组织好的每个 shard 级请求，调用处理 GET 请求时使用 TransportSingleShardAction#AsyncSingleAction处理单个doc的流程。

（3）收集Response，全部Response返回后执行finishHim（），给客户端返回结果。

回复的消息中文档顺序与请求的顺序一致。如果部分文档读取失败，则不影响其他结果，检索失败的doc会在回复信息中标出。

# Search流程
GET操作只能对单个文档进行处理，由_index、_type和_id三元组来确定唯一文档。但搜索需要一种更复杂的模型，因为不知道查询会命中哪些文档。

找到匹配文档仅仅完成了搜索流程的一半，因为多分片中的结果必须组合成单个排序列表。集群的任意节点都可以接收搜索请求，接收客户端请求的节点称为协调节点。在协调节点，搜索任务被执行成一个两阶段过程，即query then fetch。真正执行搜索任务的节点称为数据节点。

需要两个阶段才能完成搜索的原因是，在查询的时候不知道文档位于哪个分片，因此索引的所有分片（某个副本）都要参与搜索，然后协调节点将结果合并，再根据文档ID获取文档内容。例如，有5个分片，查询返回前10个匹配度最高的文档，那么每个分片都查询出当前分片的TOP 10，协调节点将5×10 = 50的结果再次排序，返回最终TOP 10的结果给客户端。

## 索引和搜索
ES中的数据可以分为两类：精确值和全文。

· 精确值，比如日期和用户id、IP地址等。

· 全文，指文本内容，比如一条日志，或者邮件的内容。

这两种类型的数据在查询时是不同的：对精确值的比较是二进制的，查询要么匹配，要么不匹配；全文内容的查询无法给出“有”还是“没有”的结果，它只能找到结果是“看起来像”你要查询的东西，因此把查询结果按相似度排序，评分越高，相似度越大。

对数据建立索引和执行搜索的原理如下图所示。

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/es_5.png" width="700" height="500">
</div>

# Search详细流程

Search由两个部分组成, Query和Fetch

**QUERY_THEN_FETCH搜索类型的查询阶段步骤如下：**

（1）客户端发送search请求到NODE 3。

（2）Node 3将查询请求转发到索引的每个主分片或副分片中。

（3）每个分片在本地执行查询，并使用本地的Term/Document Frequency信息进行打分，添加结果到大小为from + size的本地有序优先队列中。

（4）每个分片返回各自优先队列中所有文档的ID和排序值给协调节点，协调节点合并这些值到自己的优先队列中，产生一个全局排序后的列表。

协调节点广播查询请求到所有相关分片时，可以是主分片或副分片，协调节点将在之后的请求中轮询所有的分片副本来分摊负载。

查询阶段并不会对搜索请求的内容进行解析，无论搜索什么内容，只看本次搜索需要命中哪些shard，然后针对每个特定shard选择一个副本，转发搜索请求。


**Fetch阶段由以下步骤构成：**

（1）协调节点向相关NODE发送GET请求。

（2）分片所在节点向协调节点返回数据。

（3）协调节点等待所有文档被取得，然后返回给客户端。

分片所在节点在返回文档数据时，处理有可能出现的_source字段和高亮参数。

协调节点首先决定哪些文档“确实”需要被取回，例如，如果查询指定了{ ＂from＂: 90, ＂size＂:10 }，则只有从第91个开始的10个结果需要被取回。

为了避免在协调节点中创建的number_of_shards * （from + size）优先队列过大，应尽量控制分页深度。

[blog](https://blog.csdn.net/u010454030/article/details/79794788)

# 索引恢复流程分析
索引恢复（indices.recovery）是ES数据恢复过程。待恢复的数据是客户端写入成功，但未执行刷盘（flush）的Lucene分段。例如，当节点异常重启时，写入磁盘的数据先到文件系统的缓冲，未必来得及刷盘，如果不通过某种方式将未刷盘的数据找回来，则会丢失一些数据，这是保持数据完整性的体现；另一方面，由于写入操作在多个分片副本上没有来得及全部执行，副分片需要同步成和主分片完全一致，这是数据副本一致性的体现。

根据数据分片性质，索引恢复过程可分为主分片恢复流程和副分片恢复流程。

* 主分片从translog中自我恢复，尚未执行flush到磁盘的Lucene分段可以从translog中重建；

* 副分片需要从主分片中拉取Lucene分段和translog进行恢复。但是有机会跳过拉取Lucene分段的过程。

索引恢复的触发条件包括从快照备份恢复、节点加入和离开、索引的_open操作等。

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/es_6.png" width="500" height="300">
</div>

# Allocation
分片分配就是把一个分片指派到集群中某个节点的过程。分配决策由主节点完成，分配决策包含两方面：

· 哪些分片应该分配给哪些节点；

· 哪个分片作为主分片，哪些作为副分片。

对于新建索引和已有索引，分片分配过程也不尽相同。不过不管哪种场景，ES都通过两个基础组件完成工作：allocators和deciders。allocators尝试寻找最优的节点来分配分片，deciders则负责判断并决定是否要进行这次分配。

· 对于新建索引，allocators负责找出拥有分片数最少的节点列表，并按分片数量升序排序，因此分片较少的节点会被优先选择。所以对于新建索引，allocators的目标就是以更均衡的方式把新索引的分片分配到集群的节点中。然后deciders依次遍历allocators给出的节点，并判断是否把分片分配到该节点。例如，如果分配过滤规则中禁止节点A持有索引idx中的任一分片，那么过滤器也阻止把索引idx分配到节点A中，即便A节点是allocators从集群负载均衡角度选出的最优节点。需要注意的是，allocators只关心每个节点上的分片数，而不管每个分片的具体大小。这恰好是 deciders 工作的一部分，即阻止把分片分配到将超出节点磁盘容量阈值的节点上。

· 对于已有索引，则要区分主分片还是副分片。对于主分片，allocators只允许把主分片指定在已经拥有该分片完整数据的节点上。而对于副分片，allocators则是先判断其他节点上是否已有该分片的数据的副本（即便数据不是最新的）。如果有这样的节点，则allocators优先把分片分配到其中一个节点。因为副分片一旦分配，就需要从主分片中进行数据同步，所以当一个节点只拥分片中的部分数据时，也就意味着那些未拥有的数据必须从主节点中复制得到。这样可以明显地提高副分片的数据恢复速度。

## 触发时机
触发分片分配有以下几种情况：

· index增删；

· node增删；

· 手工reroute；

· replica数量改变；

· 集群重启。

# Shrink
索引分片数量一般在模板中统一定义，在数据规模比较大的索引中，索引分片数一般也大一些，在笔者的集群中设置为24。同时按天生成新的索引，使用别名关联。但是，并非每天的索引数据量都很大，小数据量的索引同样有较大的分片数。在 ES 中，主节点管理分片是很大的工作量，降低集群整体分片数量可以减少recovery时间，减小集群状态的大小。因此，可以使用Shrink API缩小索引分片数。当索引缩小完成后，源索引可以删除。

Shrink API是ES 5.0之后提供的新功能，其可以缩小主分片数量。但其并不对源索引直接进行缩小操作，而是使用与源索引相同的配置创建一个新索引，仅降低分片数。由于添加新文档时使用对分片数量取余获取目的分片的关系，新索引的主分片数必须是源索引主分片数的因数。例如，8个分片可以缩小到4、2、1个分片。如果源索引的分片数为素数，则目标索引的分片数只能为1。

## 原理
引用官方手册对Shrink工作过程的描述：

· 以相同配置创建目标索引，但是降低主分片数量。

· 从源索引的Lucene分段创建硬链接到目的索引。如果系统不支持硬链接，那么索引的所有分段都将复制到新索引，将会花费大量时间。

· 对目标索引执行恢复操作，就像一个关闭的索引重新打开时一样。

# 写入速度优化
在 ES 的默认设置下，是综合考虑数据可靠性、搜索实时性、写入速度等因素的。当离开默认设置、追求极致的写入速度时，很多是以牺牲可靠性和搜索实时性为代价的。有时候，业务上对数据可靠性和搜索实时性要求并不高，反而对写入速度要求很高，此时可以调整一些策略，最大化写入速度。

接下来的优化基于集群正常运行的前提下，如果是集群首次批量导入数据，则可以将副本数设置为0，导入完毕再将副本数调整回去，这样副分片只需要复制，节省了构建索引过程。

综合来说，提升写入速度从以下几方面入手：

· 加大translog flush间隔，目的是降低iops、writeblock。

· 加大index refresh间隔，除了降低I/O，更重要的是降低了segment merge频率。

· 调整bulk请求。

· 优化磁盘间的任务均匀情况，将shard尽量均匀分布到物理主机的各个磁盘。

· 优化节点间的任务分布，将任务尽量均匀地发到各节点。

· 优化Lucene层建立索引的过程，目的是降低CPU占用率及I/O，例如，禁用_all字段。



## translog flush间隔调整
从ES 2.x开始，在默认设置下，translog的持久化策略为：每个请求都“flush”。对应配置项如下：

index.translog.durability: request

这是影响 ES 写入速度的最大因素。但是只有这样，写操作才有可能是可靠的。如果系统可以接受一定概率的数据丢失（例如，数据写入主分片成功，尚未复制到副分片时，主机断电。由于数据既没有刷到Lucene,translog也没有刷盘，恢复时translog中没有这个数据，数据丢失），则调整translog持久化策略为周期性和一定大小的时候“flush”，例如：

index.translog.durability: async

设置为async表示translog的刷盘策略按sync_interval配置指定的时间周期进行。

index.translog.sync_interval: 120s

加大translog刷盘间隔时间。默认为5s，不可低于100ms。

index.translog.flush_threshold_size: 1024mb

超过这个大小会导致refresh操作，产生新的Lucene分段。默认值为512MB。

## 索引刷新间隔refresh_interval
默认情况下索引的refresh_interval为1秒，这意味着数据写1秒后就可以被搜索到，每次索引的refresh会产生一个新的Lucene段，这会导致频繁的segment merge行为，如果不需要这么高的搜索实时性，应该降低索引refresh周期，例如：

index.refresh_interval: 120s

##  索引过程调整和优化
### 自动生成doc ID
通过ES写入流程可以看出，写入doc时如果外部指定了id，则ES会先尝试读取原来doc的版本号，以判断是否需要更新。这会涉及一次读取磁盘的操作，通过自动生成doc ID可以避免这个环节。

### 调整字段Mappings
（1）减少字段数量，对于不需要建立索引的字段，不写入ES。

（2）将不需要建立索引的字段index属性设置为not_analyzed或no。对字段不分词，或者不索引，可以减少很多运算操作，降低CPU占用。尤其是binary类型，默认情况下占用CPU非常高，而这种类型进行分词通常没有什么意义。

（3）减少字段内容长度，如果原始数据的大段内容无须全部建立索引，则可以尽量减少不必要的内容。

（4）使用不同的分析器（analyzer），不同的分析器在索引过程中运算复杂度也有较大的差异。

### 调整_source字段
_source 字段用于存储 doc 原始数据，对于部分不需要存储的字段，可以通过 includes excludes过滤，或者将_source禁用，一般用于索引和数据分离。

这样可以降低 I/O 的压力，不过实际场景中大多不会禁用_source，而即使过滤掉某些字段，对于写入速度的提升作用也不大，满负荷写入情况下，基本是 CPU 先跑满了，瓶颈在于CPU。

### 禁用_all字段
从ES 6.0开始，_all字段默认为不启用，而在此前的版本中，_all字段默认是开启的。_all字段中包含所有字段分词后的关键词，作用是可以在搜索的时候不指定特定字段，从所有字段中检索。ES 6.0默认禁用_all字段主要有以下几点原因：

· 由于需要从其他的全部字段复制所有字段值，导致_all字段占用非常大的空间。

· _all 字段有自己的分析器，在进行某些查询时（例如，同义词），结果不符合预期，因为没有匹配同一个分析器。

· 由于数据重复引起的额外建立索引的开销。

· 想要调试时，其内容不容易检查。

· 有些用户甚至不知道存在这个字段，导致了查询混乱。

· 有更好的替代方法。

关于这个问题的更多讨论可以参考https://github.com/elastic/elasticsearch/issues/19784。

在ES 6.0之前的版本中，可以在mapping中将enabled设置为false来禁用_all字段：

curl -X PUT ＂localhost:9200/my_index＂ -H ＇Content-Type: application/json＇ -d＇

{

＂mappings＂: {

＂type_1＂: {

＂_all＂: {

＂enabled＂: false

}，

＂properties＂: {...}

}

}

}

＇

禁用_all字段可以明显降低对CPU和I/O的压力。

### 对Analyzed的字段禁用Norms
Norms用于在搜索时计算doc的评分，如果不需要评分，则可以将其禁用：

＂title＂: {＂type＂: ＂string＂,＂norms＂: {＂enabled＂: false}}

### index_options 设置
index_options用于控制在建立倒排索引过程中，哪些内容会被添加到倒排索引，例如，doc数量、词频、positions、offsets等信息，优化这些设置可以一定程度降低索引过程中的运算任务，节省CPU占用率。

不过在实际场景中，通常很难确定业务将来会不会用到这些信息，除非一开始方案就明确是这样设计的。

## 思考与总结
（1）方法比结论重要。一个系统性问题往往是多种因素造成的，在处理集群的写入性能问题上，先将问题分解，在单台上进行压测，观察哪种系统资源达到极限，例如，CPU或磁盘利用率、I/O block、线程切换、堆栈状态等。然后分析并调整参数，优化单台上的能力，先解决局部问题，在此基础上解决整体问题会容易得多。

（2）可以使用更好的 CPU，或者使用 SSD，对写入性能提升明显。在我们的测试中，在相同条件下，E5 2650V4比E5 2430 v2的写入速度高60%左右。

（3）在我们的压测环境中，写入速度稳定在平均单机每秒3万条以上，使用的测试数据：每个文档的字段数量为10个左右，文档大小约100字节，CPU使用E5 2430 v2。

# ES搜索速度优化
## 增加Cache, 硬件优化

## 文档模型






# 问
1. 什么情况下es数据会丢失