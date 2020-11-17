### [Streams and Monk – How Yelp is Approaching Kafka in 2020](https://engineeringblog.yelp.com/2020/01/streams-and-monk-how-yelp-approaches-kafka-in-2020.html)

将不同的stream分类, 分为紧急非紧急, 从而根据不同情况来部署cluster

另外创建topic也是根据用户指定的信息来在不同的cluser上创建

之后用到了一个叫Monk的unified stream ingestion pipeline, 对于fire-forget, 每次produce信息, 都是先produce到一个1M的memory (disk, disk可以避免信息丢失)中, 由其他的工作线程来读取然后push到topic里, 相当于一个缓冲, 避免overload

### More Than Just a Schema Store
This post is part of a series covering Yelp's real-time streaming data infrastructure. Our series explores in-depth how we stream MySQL and Cassandra data at real-time, how we automatically track & migrate schemas, how we process and transform streams, and finally how we connect all of this into data stores like Redshift, Salesforce, and Elasticsearch.
用avro建立schema, topic对用户不可见

### [Billions of Messages a Day - Yelp's Real-time Data Pipeline](https://engineeringblog.yelp.com/2016/07/billions-of-messages-a-day-yelps-real-time-data-pipeline.html)
服务太多, 互相通信困难, 因此需要kafka

通信format选择json, 会导致一旦json改变, downstream的服务全部会被影响

Apache Avro, a data serialization system, has some really nice properties, and is ultimately what we selected. Avro is a space-efficient binary serialization format that integrates nicely with dynamic languages like Python, without requiring code generation. The killer feature of Avro, for our system, is that it supports schema evolution. That means that a reader application and a writer application can use different schema versions to consume and produce data, as long as the two are compatible. This decouples consumers and producers nicely - producers can iterate on their data format, without requiring changes in consumer applications.

他们提供了一个Schematizer供用户注册schema

### [Streaming MySQL tables in real-time to Kafka](https://engineeringblog.yelp.com/2016/08/streaming-mysql-tables-in-real-time-to-kafka.html)
mysql主从复制原理:
In order for replication to work, events on the master database cluster are written to a special log called the binary log. When a replica connects to its master, this binary log is read by the replica to complete or continue the process of replication, depending on the replication hierarchy. This process is bolstered by two threads running on the replica, an IO thread and a SQL thread. This is visualized in the figure below. The IO thread is primarily responsible for reading the binary log events from master, as they arrive, and copying them over to a local relay log in the replica. The SQL thread then reads these events and replays them in the same order that they arrived in.

