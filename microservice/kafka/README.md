# Kafka
We’ve come to think of Kafka as a streaming platform: a system that lets you publish and subscribe to streams of data, store them, and process them. 

### Kafka相对于其他Message queue的优点
Kafka is like a messaging system in that it lets you publish and subscribe to streams of messages. In this way, it is similar to products like ActiveMQ, RabbitMQ, IBM’s MQSeries, and other products. But even with these similarities, Kafka has a number of core differences from traditional messaging systems that make it another kind of animal entirely. Here are the big three differences: first, it works as a modern dis‐ tributed system that runs as a cluster and can scale to handle all the applications in even the most massive of companies. Rather than running dozens of individual mes‐ saging brokers, hand wired to different apps, this lets you have a central platform that can scale elastically to handle all the streams of data in a company. Secondly, Kafka is a true storage system built to store data for as long as you might like. This has huge advantages in using it as a connecting layer as it provides real delivery guarantees—its data is replicated, persistent, and can be kept around as long as you like. Finally, the world of stream processing raises the level of abstraction quite significantly. Messag‐ ing systems mostly just hand out messages. The stream processing capabilities in Kafka let you compute derived streams and datasets dynamically off of your streams with far less code. These differences make Kafka enough of its own thing that it doesn’t really make sense to think of it as “yet another queue.”

Data within Kafka is stored durably, in order, and can be read deterministically. In addition, the data can be distributed within the system to provide additional protections against failures, as well as signifi‐ cant opportunities for scaling performance.

### Messages and Batches
The unit of data within Kafka is called a message.

For efficiency, messages are written into Kafka in batches. A batch is just a collection of messages, all of which are being produced to the same topic and partition. 

### Schemas
While messages are opaque byte arrays to Kafka itself, it is recommended that addi‐ tional structure, or schema, be imposed on the message content so that it can be easily understood. There are many options available for message schema, depending on your application’s individual needs. Simplistic systems, such as Javascript Object Notation (JSON) and Extensible Markup Language (XML), are easy to use and human-readable. However, they lack features such as robust type handling and com‐ patibility between schema versions. Many Kafka developers favor the use of Apache Avro, which is a serialization framework originally developed for Hadoop. Avro pro‐ vides a compact serialization format; schemas that are separate from the message pay‐ loads and that do not require code to be generated when they change; and strong data typing and schema evolution, with both backward and forward compatibility.

### Topics and Partitions
**Messages** in Kafka are categorized into topics. The closest analogies for a topic are a database table or a folder in a filesystem. Topics are additionally broken down into a number of partitions. Going back to the “commit log” description, a **partition** is a single log. Messages are written to it in an append-only fashion, and are read in order from beginning to end. 

The term **stream** is often used when discussing data within systems like Kafka. Most often, a stream is considered to be a single topic of data, regardless of the number of partitions. This represents a single stream of data moving from the producers to the consumers. This way of referring to messages is most common when discussing stream processing, which is when frameworks—some of which are Kafka Streams,

### Producers and Consumers
**Producers** create new messages. In other publish/subscribe systems, these may be called publishers or writers. In general, a message will be produced to a specific topic. By default, the producer does not care what partition a specific message is written to and will balance messages over all partitions of a topic evenly. In some cases, the pro‐ ducer will direct messages to specific partitions. This is typically done using the mes‐ sage key and a partitioner that will generate a hash of the key and map it to a specific partition. This assures that all messages produced with a given key will get written to the same partition. The producer could also use a custom partitioner that follows other business rules for mapping messages to partitions. 

Consumers read messages. In other publish/subscribe systems, these clients may be called subscribers or readers. The consumer subscribes to one or more topics and reads the messages in the order in which they were produced. The consumer keeps track of which messages it has already consumed by keeping track of the offset of messages. The offset is another bit of metadata—an integer value that continually increases—that Kafka adds to each message as it is produced. Each message in a given partition has a unique offset. By storing the offset of the last consumed message for each partition, either in Zookeeper or in Kafka itself, a consumer can stop and restart without losing its place.

### Brokers and Clusters
A single Kafka server is called a broker. The broker receives messages from producers, assigns offsets to them, and commits the messages to storage on disk. It also services consumers, responding to fetch requests for partitions and responding with the mes‐ sages that have been committed to disk. Depending on the specific hardware and its performance characteristics, a single broker can easily handle thousands of partitions and millions of messages per second.

Kafka brokers are designed to operate as part of a cluster. 

## Use Cases

* Activity Tracking
* Messaging
* Metrics and logging
* Commit log
* Stream processing