[TOC]

# Kafka
We’ve come to think of Kafka as a streaming platform: a system that lets you publish and subscribe to streams of data, store them, and process them. 

# Why Kafka
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_1.png" width="500" height="300">
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_2.png" width="500" height="300">
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_3.png" width="500" height="300">
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_4.png" width="500" height="300">
</div>

### Kafka相对于其他Message queue的优点
Kafka is like a messaging system in that it lets you publish and subscribe to streams of messages. In this way, it is similar to products like ActiveMQ, RabbitMQ, IBM’s MQSeries, and other products. But even with these similarities, Kafka has a number of core differences from traditional messaging systems that make it another kind of animal entirely. Here are the big **three differences**: first, it works as a modern distributed system that runs as a cluster and can scale to handle all the applications in even the most massive of companies. Rather than running dozens of individual messaging brokers, hand wired to different apps, this lets you have a central platform that can scale elastically to handle all the streams of data in a company. Secondly, Kafka is a true storage system built to store data for as long as you might like. This has huge advantages in using it as a connecting layer as it provides real delivery guarantees—its data is replicated, persistent, and can be kept around as long as you like. Finally, the world of stream processing raises the level of abstraction quite significantly. Messaging systems mostly just hand out messages. The stream processing capabilities in Kafka let you compute derived streams and datasets dynamically off of your streams with far less code. These differences make Kafka enough of its own thing that it doesn’t really make sense to think of it as “yet another queue.”

Data within Kafka is stored durably, in order, and can be read deterministically. In addition, the data can be distributed within the system to provide additional protections against failures, as well as significant opportunities for scaling performance.

### Messages and Batches
The unit of data within Kafka is called a message.

For efficiency, messages are written into Kafka in batches. A batch is just a collection of messages, all of which are being produced to the same topic and partition. 

### Schemas
While messages are opaque byte arrays to Kafka itself, it is recommended that additional structure, or schema, be imposed on the message content so that it can be easily understood. There are many options available for message schema, depending on your application’s individual needs. Simplistic systems, such as Javascript Object Notation (JSON) and Extensible Markup Language (XML), are easy to use and human-readable. However, they lack features such as robust type handling and com‐ patibility between schema versions. Many Kafka developers favor the use of Apache Avro, which is a serialization framework originally developed for Hadoop. Avro provides a compact serialization format; schemas that are separate from the message pay‐ loads and that do not require code to be generated when they change; and strong data typing and schema evolution, with both backward and forward compatibility.

### Topics and Partitions
**Messages** in Kafka are categorized into topics. The closest analogies for a topic are a database table or a folder in a filesystem. Topics are additionally broken down into a number of partitions. Going back to the “commit log” description, a **partition** is a single log. Messages are written to it in an append-only fashion, and are read in order from beginning to end. Partitions are also the way that Kafka provides redundancy and scalability. Each partition can be hosted on a different server, which means that a single topic can be scaled horizontally across multiple servers to provide performance far beyond the ability of a single server.

The term **stream** is often used when discussing data within systems like Kafka. Most often, a stream is considered to be a single topic of data, regardless of the number of partitions. This represents a single stream of data moving from the producers to the consumers. This way of referring to messages is most common when discussing stream processing, which is when frameworks—some of which are Kafka Streams,

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_writemessage.png" width="500" height="300">
</div>

### Producers and Consumers
**Producers** create new messages. In other publish/subscribe systems, these may be called publishers or writers. In general, a message will be produced to a specific topic. By default, the producer does not care what partition a specific message is written to and will balance messages over all partitions of a topic evenly. In some cases, the producer will direct messages to specific partitions. **This is typically done using the message key and a partitioner that will generate a hash of the key and map it to a specific partition.** This assures that all messages produced with a given key will get written to the same partition. The producer could also use a custom partitioner that follows other business rules for mapping messages to partitions. 

**Consumers** read messages. In other publish/subscribe systems, these clients may be called subscribers or readers. The consumer subscribes to one or more topics and reads the messages in the order in which they were produced. The consumer keeps track of which messages it has already consumed by keeping track of the offset of messages. The offset is another bit of metadata—an integer value that continually increases—that Kafka adds to each message as it is produced. Each message in a given partition has a unique offset. By storing the offset of the last consumed message for each partition, either in Zookeeper or in Kafka itself, a consumer can stop and restart without losing its place.
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_readmessage.png" width="500" height="300">
</div>

### Brokers and Clusters
A single Kafka server is called a **broker**. The broker receives messages from producers, assigns offsets to them, and commits the messages to storage on disk. It also services consumers, responding to fetch requests for partitions and responding with the messages that have been committed to disk. Depending on the specific hardware and its performance characteristics, a single broker can easily handle thousands of partitions and millions of messages per second.

Kafka brokers are designed to operate as part of a cluster. 

A key feature of Apache Kafka is that of retention, which is the durable storage of messages for some period of time. Kafka brokers are configured with a default reten‐ tion setting for topics, either retaining messages for some period of time (e.g., 7 days) or until the topic reaches a certain size in bytes.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_brokers.png" width="500" height="300">
</div>

## Why Kafka
1. **Multiple Producers** Kafka is able to seamlessly handle multiple producers, whether those clients are using many topics or the same topic. This makes the system ideal for aggregating data from many frontend systems and making it consistent. 
2. **Multiple Consumers** In addition to multiple producers, Kafka is designed for multiple consumers to read any single stream of messages without interfering with each other. This is in contrast to many queuing systems where once a message is consumed by one client, it is not available to any other. Multiple Kafka consumers can choose to operate as part of a group and share a stream, assuring that the entire group processes a given message only once.
3. **Disk-Based Retention** Not only can Kafka handle multiple consumers, but durable message retention means that consumers do not always need to work in real time. Messages are committed to disk, and will be stored with configurable retention rules. These options can be selected on a per-topic basis, allowing for different streams of messages to have differ‐ ent amounts of retention depending on the consumer needs. Durable retention means that if a consumer falls behind, either due to slow processing or a burst in traf‐ fic, there is no danger of losing data. It also means that maintenance can be per‐ formed on consumers, taking applications offline for a short period of time, with no concern about messages backing up on the producer or getting lost. Consumers can be stopped, and the messages will be retained in Kafka. This allows them to restart and pick up processing messages where they left off with no data loss.
4. **Scalable** Kafka’s flexible scalability makes it easy to handle any amount of data. Users can start with a single broker as a proof of concept, expand to a small development cluster of three brokers, and move into production with a larger cluster of tens or even hun‐ dreds of brokers that grows over time as the data scales up. Expansions can be performed while the cluster is online, with no impact on the availability of the system as a whole. This also means that a cluster of multiple brokers can handle the failure of an individual broker, and continue servicing clients.
5. **High Performance**

## Use Cases
* **Activity Tracking**
The original use case for Kafka, as it was designed at LinkedIn, is that of user activity tracking. A website’s users interact with frontend applications, which generate mes‐ sages regarding actions the user is taking. This can be passive information, such as page views and click tracking, or it can be more complex actions, such as information that a user adds to their profile. The messages are published to one or more topics, which are then consumed by applications on the backend. These applications may be generating reports, feeding machine learning systems, updating search results, or per‐ forming other operations that are necessary to provide a rich user experience.
* Messaging
Kafka is also used for messaging, where applications need to send notifications (such as emails) to users. Those applications can produce messages without needing to be concerned about formatting or how the messages will actually be sent. A single appli‐ cation can then read all the messages to be sent and handle them consistently, including:
• Formatting the messages (also known as decorating) using a common look and feel

• Collecting multiple messages into a single notification to be sent

• Applying a user’s preferences for how they want to receive messages

Using a single application for this avoids the need to duplicate functionality in multi‐ ple applications, as well as allows operations like aggregation which would not other‐ wise be possible.
* **Metrics and logging**
Kafka is also ideal for collecting application and system metrics and logs. This is a use case in which the ability to have multiple applications producing the same type of message shines. Applications publish metrics on a regular basis to a Kafka topic, and those metrics can be consumed by systems for monitoring and alerting. They can also be used in an offline system like Hadoop to perform longer-term analysis, such as growth projections. Log messages can be published in the same way, and can be routed to dedicated log search systems like Elastisearch or security analysis applica‐ tions. Another added benefit of Kafka is that when the destination system needs to change (e.g., it’s time to update the log storage system), there is no need to alter the frontend applications or the means of aggregation.

* **Commit log**
* **Stream processing**

# Producer
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_producer.png" width="700" height="500">
</div>
We start producing messages to Kafka by creating a **ProducerRecord**, which must include the topic we want to send the record to and a value. Optionally, we can also specify a key and/or a partition. Once we send the ProducerRecord, the first thing the producer will do is serialize the key and value objects to ByteArrays so they can be sent over the network.

Next, the data is sent to a **partitioner**. If we specified a partition in the ProducerRecord, the partitioner doesn’t do anything and simply returns the partition we specified. If we didn’t, the partitioner will choose a partition for us, usually based on the ProducerRecord key. Once a partition is selected, the producer knows which topic and partition the record will go to. It then adds the record to a batch of records that will also be sent to the same topic and partition. A separate thread is responsible for sending those batches of records to the appropriate Kafka brokers.

When the broker receives the messages, it sends back a response. If the messages were successfully written to Kafka, it will return a RecordMetadata object with the topic, partition, and the offset of the record within the partition. If the broker failed to write the messages, it will return an error. When the producer receives an error, it may retry sending the message a few more times before giving up and returning an error.

### Constructing a Kafka Producer
The first step in writing messages to Kafka is to create a producer object with the properties you want to pass to the producer. A Kafka producer has three mandatory properties:
#### bootstrap.servers
List of host:port pairs of brokers that the producer will use to establish initial connection to the Kafka cluster. This list doesn’t need to include all brokers, since the producer will get more information after the initial connection. But it is rec‐ ommended to include at least two, so in case one broker goes down, the producer will still be able to connect to the cluster.
#### key.serializer
Name of a class that will be used to serialize the keys of the records we will pro‐ duce to Kafka. Kafka brokers expect byte arrays as keys and values of messages. However, the producer interface allows, using parameterized types, any Java object to be sent as a key and value. This makes for very readable code, but it also means that the producer has to know how to convert these objects to byte arrays. key.serializer should be set to a name of a class that implements the org.apache.kafka.common.serialization.Serializer interface. The producer will use this class to serialize the key object to a byte array. The Kafka client pack‐ age includes ByteArraySerializer (which doesn’t do much), StringSerializer, and IntegerSerializer, so if you use common types, there is no need to implement your own serializers. Setting key.serializer is required even if you intend to send only values.
#### value.serializer
Name of a class that will be used to serialize the values of the records we will pro‐ duce to Kafka. The same way you set key.serializer to a name of a class that will serialize the message key object to a byte array, you set value.serializer to a class that will serialize the message value object.

```java
private Properties kafkaProps = new Properties();
kafkaProps.put("bootstrap.servers", "broker1:9092,broker2:9092");
kafkaProps.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
kafkaProps.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
//props.put("key.serializer","org.apache.kafka.common.serialization.StringDeserializer");
//props.put("value.serializer","io.confluent.kafka.serializers.KafkaAvroDeserializer");
producer = new KafkaProducer<String, String>(kafkaProps);
```

Once we instantiate a producer, it is time to start sending messages. There are three primary methods of sending messages:
* Fire-and-forget We send a message to the server and don’t really care if it arrives succesfully or not. Most of the time, it will arrive successfully, since Kafka is highly available and the producer will retry sending messages automatically. However, some messages will get lost using this method.
* Synchronous send We send a message, the send() method returns a Future object, and we use get() to wait on the future and see if the send() was successful or not.
* Asynchronous send We call the send() method with a callback function, which gets triggered when it receives a response from the Kafka broker.

## Sending a Message to Kafka
```java
ProducerRecord<String, String> record =
            new ProducerRecord<>("CustomerCountry", "Precision Products",
"France");
//topic key value
    try {
      producer.send(record);    //ignore the result
    } catch (Exception e) {
            e.printStackTrace();    //Those can be a SerializationException when it fails to serialize the message, a BufferExhaus tedException or TimeoutException if the buffer is full, or an InterruptException if the sending thread was interrupted.
}
```

### Sending a Message Synchronously
```java
 ProducerRecord<String, String> record =
            new ProducerRecord<>("CustomerCountry", "Precision Products", "France");
    try {
            producer.send(record).get();
            /*Here, we are using Future.get() to wait for a reply from Kafka. This method will throw an exception if the record is not sent successfully to Kafka. If there were no errors, we will get a RecordMetadata object that we can use to retrieve the offset the message was written to.
            */
    } catch (Exception e) {
e.printStackTrace();
}
```
KafkaProducer has two types of errors. Retriable errors are those that can be resolved by sending the message again. For example, a connection error can be resolved because the connection may get reestablished. A “no leader” error can be resolved when a new leader is elected for the partition. KafkaProducer can be configured to retry those errors automatically, so the application code will get retriable exceptions only when the number of retries was exhausted and the error was not resolved. Some errors will not be resolved by retrying. For example, “message size too large.” In those cases, KafkaProducer will not attempt a retry and will return the exception immedi‐ ately.

### Sending a Message Asynchronously
```java
private class DemoProducerCallback implements Callback {
            @Override
        public void onCompletion(RecordMetadata recordMetadata, Exception e) {
         if (e != null) {
             e.printStackTrace();
            }
} }

ProducerRecord<String, String> record =
        new ProducerRecord<>("CustomerCountry", "Biomedical Materials", "USA");
producer.send(record, new DemoProducerCallback());
```

### 一些常用参数
**acks**

The acks parameter controls how many partition replicas must receive the record before the producer can consider the write successful. If **acks=0**, the producer will not wait for a reply from the broker before assuming the message was sent successfully. If **acks=1**, the producer will receive a success response from the broker the moment the leader replica received the message. If **acks=all**, the producer will receive a success response from the broker once all in-sync replicas received the message.

**buffer.memory**

This sets the amount of memory the producer will use to buffer messages waiting to be sent to brokers. If messages are sent by the application faster than they can be delivered to the server, the producer may run out of space. 

**retries**

**batch.size**

**linger.ms**
linger.ms controls the amount of time to wait for additional messages before send‐ ing the current batch. KafkaProducer sends a batch of messages either when the cur‐ rent batch is full or when the linger.ms limit is reached. By default, the producer will send messages as soon as there is a sender thread available to send them, even if there’s just one message in the batch. By setting linger.ms higher than 0, we instruct the producer to wait a few milliseconds to add additional messages to the batch before sending it to the brokers. This increases latency but also increases throughput

**client.id**
This can be any string, and will be used by the brokers to identify messages sent from the client. It is used in logging and metrics, and for quotas.

**max.in.flight.requests.per.connection**

This controls how many messages the producer will send to the server without receiving responses. Setting this high can increase memory usage while improving throughput, but setting it too high can reduce throughput as batching becomes less efficient. Setting this to 1 will guarantee that messages will be written to the broker in the order in which they were sent, even when retries occur.

**timeout.ms, request.timeout.ms, and metadata.fetch.timeout.ms**

These parameters control how long the producer will wait for a reply from the server when sending data (request.timeout.ms) and when requesting metadata such as the current leaders for the partitions we are writing to (metadata.fetch.timeout.ms). If the timeout is reached without reply, the producer will either retry sending or respond with an error (either through exception or the send callback). timeout.ms controls the time the broker will wait for in-sync replicas to acknowledge the message in order to meet the acks configuration—the broker will return an error if the time elapses without the necessary acknowledgments.

**max.block.ms**

This parameter controls how long the producer will block when calling send() and when explicitly requesting metadata via partitionsFor(). Those methods block when the producer’s send buffer is full or when metadata is not available. When max.block.ms is reached, a timeout exception is thrown.

**max.request.size**

It caps both the size of the largest message that can be sent and the number of messages that the producer can send in one request. 

**receive.buffer.bytes and send.buffer.bytes**

These are the sizes of the TCP send and receive buffers used by the sockets when writing and reading data. If these are set to -1, the OS defaults will be used. 

## Serializers
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_schema_register.png" width="700" height="500">
</div>

```java
 Properties props = new Properties();
    props.put("bootstrap.servers", "localhost:9092");
    props.put("key.serializer",
       "io.confluent.kafka.serializers.KafkaAvroSerializer");
    props.put("value.serializer",
       "io.confluent.kafka.serializers.KafkaAvroSerializer");
    props.put("schema.registry.url", schemaUrl);
    String topic = "customerContacts";
    int wait = 500;
    Producer<String, Customer> producer = new KafkaProducer<String,
       Customer>(props);
    // We keep producing new events until someone ctrl-c
    while (true) {
        Customer customer = CustomerGenerator.getNext();
        System.out.println("Generated customer " +
           customer.toString());
        ProducerRecord<String, Customer> record =
                            new ProducerRecord<>(topic, customer.getId(), cus-
    tomer);
        producer.send(record);
}
```

What if you prefer to use generic Avro objects rather than the generated Avro objects? No worries. In this case, you just need to provide the schema:

```java

    Properties props = new Properties();
    props.put("bootstrap.servers", "localhost:9092");
    props.put("key.serializer",
       "io.confluent.kafka.serializers.KafkaAvroSerializer");
    props.put("value.serializer",
       "io.confluent.kafka.serializers.KafkaAvroSerializer");
    props.put("schema.registry.url", url);
        String schemaString = "{\"namespace\": \"customerManagement.avro\",
    \"type\": \"record\", " +
                               "\"name\": \"Customer\"," +
                               "\"fields\": [" +
                                "{\"name\": \"id\", \"type\": \"int\"}," +
                                "{\"name\": \"name\", \"type\": \"string\"}," +
                                "{\"name\": \"email\", \"type\": [\"null\",\"string
    \"], \"default\":\"null\" }" +
                               "]}";
        Producer<String, GenericRecord> producer =
           new KafkaProducer<String, GenericRecord>(props);
        Schema.Parser parser = new Schema.Parser();
        Schema schema = parser.parse(schemaString);
        for (int nCustomers = 0; nCustomers < customers; nCustomers++) {
          String name = "exampleCustomer" + nCustomers;
          String email = "example " + nCustomers + "@example.com"
          GenericRecord customer = new GenericData.Record(schema);
          customer.put("id", nCustomer);
          customer.put("name", name);
          customer.put("email", email);
          ProducerRecord<String, GenericRecord> data =
                                         new ProducerRecord<String,
                                            GenericRecord>("customerContacts",
    name, customer);
          producer.send(data);
          } }
```

## Partitions
In previous examples, the ProducerRecord objects we created included a topic name, key, and value. Kafka messages are key-value pairs and while it is possible to create a ProducerRecord with just a topic and a value, with the key set to null by default, most applications produce records with keys. Keys serve two goals: they are addi‐ tional information that gets stored with the message, and they are also used to decide which one of the topic partitions the message will be written to. All messages with the same key will go to the same partition. This means that if a process is reading only a subset of the partitions in a topic (more on that in Chapter 4), all the records for a single key will be read by the same process.

When the key is null and the default partitioner is used, the record will be sent to one of the available partitions of the topic at random. A round-robin algorithm will be used to balance the messages among the partitions.

If a key exists and the default partitioner is used, Kafka will hash the key (using its own hash algorithm, so hash values will not change when Java is upgraded), and use the result to map the message to a specific partition. Since it is important that a key is always mapped to the same partition, we use all the partitions in the topic to calculate the mapping—not just the available partitions. This means that if a specific partition is unavailable when you write data to it, you might get an error. 

The mapping of keys to partitions is consistent only as long as the number of parti‐ tions in a topic does not change. So as long as the number of partitions is constant, you can be sure that, for example, records regarding user 045189 will always get writ‐ ten to partition 34. This allows all kinds of optimization when reading data from par‐ titions. However, the moment you add new partitions to the topic, this is no longer guaranteed—the old records will stay in partition 34 while new records will get writ‐ ten to a different partition. When partitioning keys is important, the easiest solution is to create topics with sufficient partitions (Chapter 2 includes suggestions for how to determine a good number of partitions) and never add partitions.


# Creating a Kafka Consumer
Producer's rate maybe exceed the rate that the consumer read and process data. Just like multiple producers can write to the same topic, we need to allow multiple consumers to read from the same topic, splitting the data between them.

Kafka consumers are typically part of a **consumer group**. When multiple consumers are subscribed to a topic and belong to the same consumer group, each consumer in the group will receive messages from a different subset of the partitions in the topic.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_consumer_group.png" width="700" height="1300">
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_consumer_group2.png" width="700" height="1100">
</div>

To make sure an application gets all the messages in a topic, ensure the application has its own con‐ sumer group. Unlike many traditional messaging systems, Kafka scales to a large number of consumers and consumer groups without reducing performance.

## Consumer Groups and Partition Rebalance
When we add a new consumer to the group, it starts consuming messages from partitions previously consumed by another consumer. The same thing happens when a consumer shuts down or crashes; it leaves the group, and the partitions it used to consume will be consumed by one of the remaining consumers. Reassignment of partitions to consumers also happen when the topics the consumer group is consuming are modified.

Moving partition ownership from one consumer to another is called a rebalance. Rebalances are important because they provide the consumer group with high availa‐ bility and scalability (allowing us to easily and safely add and remove consumers), but in the normal course of events they are fairly undesirable. During a rebalance, con‐ sumers can’t consume messages, so a rebalance is basically a short window of unavail‐ ability of the entire consumer group. In addition, when partitions are moved from one consumer to another, the consumer loses its current state; if it was caching any data, it will need to refresh its caches—slowing down the application until the con‐ sumer sets up its state again. Throughout this chapter we will discuss how to safely handle rebalances and how to avoid unnecessary ones.

The way consumers maintain membership in a consumer group and ownership of the partitions assigned to them is by sending heartbeats to a Kafka broker designated as the group coordinator (this broker can be different for different consumer groups). As long as the consumer is sending heartbeats at regular intervals, it is assumed to be alive, well, and processing messages from its partitions. Heartbeats are sent when the consumer polls (i.e., retrieves records) and when it commits records it has consumed.

If the consumer stops sending heartbeats for long enough, its session will time out and the group coordinator will consider it dead and trigger a rebalance. If a consumer crashed and stopped processing messages, it will take the group coordinator a few seconds without heartbeats to decide it is dead and trigger the rebalance. During those seconds, no messages will be processed from the partitions owned by the dead consumer. When closing a consumer cleanly, the consumer will notify the group coordinator that it is leaving, and the group coordinator will trigger a rebalance immediately, reducing the gap in processing. 

### How Does the Process of Assigning Partitions to Brokers Work?
When a consumer wants to join a group, it sends a JoinGroup request to the group coordinator. The first consumer to join the group becomes the group leader. The leader receives a list of all consumers in the group from the group coordinator (this will include all consumers that sent a heartbeat recently and which are therefore considered alive) and is responsible for assigning a subset of partitions to each consumer. It uses an implementation of Parti tionAssignor to decide which partitions should be handled by which consumer.
Kafka has two built-in partition assignment policies, which we will discuss in more depth in the configuration section. After deciding on the partition assignment, the consumer leader sends the list of assignments to the GroupCoordinator, which sends this informa‐ tion to all the consumers. Each consumer only sees his own assign‐ ment—the leader is the only client process that has the full list of consumers in the group and their assignments. This process repeats every time a rebalance happens.

```java
Properties props = new Properties();
props.put("bootstrap.servers", "broker1:9092,broker2:9092");
props.put("group.id", "CountryCounter");    //多了一个group id
props.put("key.deserializer",
    "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer",
    "org.apache.kafka.common.serialization.StringDeserializer");
KafkaConsumer<String, String> consumer = new KafkaConsumer<String,
String>(props);

consumer.subscribe(Collections.singletonList("customerCountries"));
consumer.subscribe("test.*");

try {
      while (true) {
          ConsumerRecords<String, String> records = consumer.poll(100);
          for (ConsumerRecord<String, String> record : records)
          {
              log.debug("topic = %s, partition = %s, offset = %d,
                 customer = %s, country = %s\n",
                 record.topic(), record.partition(), record.offset(),
                 record.key(), record.value());
              int updatedCount = 1;
              if (custCountryMap.countainsValue(record.value())) {
                  updatedCount = custCountryMap.get(record.value()) + 1;
              }
              custCountryMap.put(record.value(), updatedCount)
              JSONObject json = new JSONObject(custCountryMap);
              System.out.println(json.toString(4))
          }
}
} finally {
      consumer.close();
    }

```

The parameter we pass, poll(), is a timeout interval and controls how long poll() will block if data is not avail‐ able in the consumer buffer. If this is set to 0, poll() will return immediately; otherwise, it will wait for the specified number of milliseconds for data to arrive from the broker.

The poll loop does a lot more than just get data. The first time you call poll() with a new consumer, it is responsible for finding the GroupCoordinator, joining the con‐ sumer group, and receiving a partition assignment. If a rebalance is triggered, it will be handled inside the poll loop as well. And of course the heartbeats that keep con‐ sumers alive are sent from within the poll loop. For this reason, we try to make sure that whatever processing we do between iterations is fast and efficient.

Always **close()** the consumer before exiting. This will close the network connec‐ tions and sockets. It will also trigger a rebalance immediately rather than wait for the group coordinator to discover that the consumer stopped sending heartbeats and is likely dead, which will take longer and therefore result in a longer period of time in which consumers can’t consume messages from a subset of the parti‐ tions.

The poll loop does a lot more than just get data. The first time you call poll() with a new consumer, it is responsible for finding the GroupCoordinator, joining the con‐ sumer group, and receiving a partition assignment. If a rebalance is triggered, it will be handled inside the poll loop as well. And of course the heartbeats that keep con‐ sumers alive are sent from within the poll loop. For this reason, we try to make sure that whatever processing we do between iterations is fast and efficient.

You can’t have multiple consumers that belong to the same group in one thread and you can’t have multiple threads safely use the same consumer. One consumer per thread is the rule. To run mul‐ tiple consumers in the same group in one application, you will need to run each in its own thread. It is useful to wrap the con‐ sumer logic in its own object and then use Java’s ExecutorService to start multiple threads each with its own consumer. The Conflu‐ ent blog has a [tutorial](https://www.confluent.io/blog/tutorial-getting-started-with-the-new-apache-kafka-0-9-consumer-client/) that shows how to do just that.

## Configuring Consumers
**fetch.min.bytes**

This property allows a consumer to specify the minimum amount of data that it wants to receive from the broker when fetching records. If a broker receives a request for records from a consumer but the new records amount to fewer bytes than min.fetch.bytes, the broker will wait until more messages are available before send‐ ing the records back to the consumer.

**fetch.max.wait.ms**

By setting fetch.min.bytes, you tell Kafka to wait until it has enough data to send before responding to the consumer. fetch.max.wait.ms lets you control how long to wait. 

**max.partition.fetch.bytes**

This property controls the maximum number of bytes the server will return per parti‐ tion. The default is 1 MB, which means that when KafkaConsumer.poll() returns ConsumerRecords, the record object will use at most max.partition.fetch.bytes per partition assigned to the consumer. So if a topic has 20 partitions, and you have 5 consumers, each consumer will need to have 4 MB of memory available for Consumer Records. As you recall, the consumer must call poll() frequently enough to avoid session timeout and subsequent rebalance. If the amount of data a single poll() returns is very large, it may take the consumer longer to process, which means it will not get to the next iteration of the poll loop in time to avoid a session timeout.

**session.timeout.ms**

The amount of time a consumer can be out of contact with the brokers while still considered alive defaults to 3 seconds. If more than session.timeout.ms passes without the consumer sending a heartbeat to the group coordinator, it is considered dead and the group coordinator will trigger a rebalance of the consumer group to allocate partitions from the dead consumer to the other consumers in the group. This property is closely related to heartbeat.interval.ms. heartbeat.interval.ms con‐ trols how frequently the KafkaConsumer poll() method will send a heartbeat to the group coordinator, whereas session.timeout.ms controls how long a consumer can go without sending a heartbeat. Therefore, those two properties are typically modi‐ fied together—heatbeat.interval.ms must be lower than session.timeout.ms, and is usually set to one-third of the timeout value. So if session.timeout.ms is 3 sec‐ onds, heartbeat.interval.ms should be 1 second. Setting session.timeout.ms lower than the default will allow consumer groups to detect and recover from failure sooner, but may also cause unwanted rebalances as a result of consumers taking longer to complete the poll loop or garbage collection. Setting session.timeout.ms higher will reduce the chance of accidental rebalance, but also means it will take longer to detect a real failure.

**auto.offset.reset**

This property controls the behavior of the consumer when it starts reading a partition for which it doesn’t have a committed offset or if the committed offset it has is invalid (usually because the consumer was down for so long that the record with that offset was already aged out of the broker). The default is “latest,” which means that lacking a valid offset, the consumer will start reading from the newest records (records that were written after the consumer started running). The alternative is “earliest,” which means that lacking a valid offset, the consumer will read all the data in the partition, starting from the very beginning. 

**enable.auto.commit**

We discussed the different options for committing offsets earlier in this chapter. This parameter controls whether the consumer will commit offsets automatically, and defaults to true. Set it to false if you prefer to control when offsets are committed, which is necessary to minimize duplicates and avoid missing data. If you set enable.auto.commit to true, then you might also want to control how frequently offsets will be committed using auto.commit.interval.ms.

### Commits and Offsets
Whenever we call poll(), it returns records written to Kafka that consumers in our group have not read yet. This means that we have a way of tracking which records were read by a consumer of the group. As discussed before, one of Kafka’s unique characteristics is that it does not track acknowledgments from consumers the way many JMS queues do. Instead, it allows consumers to use Kafka to track their position (offset) in each partition.
We call the action of updating the current position in the partition a **commit**.

下一次的poll总会从offset开始, 因此会造成重复或丢失问题

```java
 try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(100);
        for (ConsumerRecord<String, String> record : records) {
            System.out.printf("topic = %s, partition = %s, offset = %d,
            customer = %s, country = %s\n",
            record.topic(), record.partition(),
            record.offset(), record.key(), record.value());
        }

        try {
          consumer.commitSync();
        } catch (CommitFailedException e) {
            log.error("commit failed", e)
        }

        //or

        consumer.commitAsync();
    }
}
```

The drawback is that while commitSync() will retry the commit until it either succeeds or encounters a nonretriable failure, commitAsync() will not retry. The reason it does not retry is that by the time commitAsync() receives a response from the server, there may have been a later commit that was already successful. Imagine that we sent a request to commit offset 2000. There is a temporary communication prob‐ lem, so the broker never gets the request and therefore never responds. Meanwhile, we processed another batch and successfully committed offset 3000. If commitA sync() now retries the previously failed commit, it might succeed in committing off‐ set 2000 after offset 3000 was already processed and committed. In the case of a rebalance, this will cause more duplicates.
We mention this complication and the importance of correct order of commits, because commitAsync() also gives you an option to pass in a callback that will be trig‐ gered when the broker responds. It is common to use the callback to log commit errors or to count them in a metric, but if you want to use the callback for retries, you need to be aware of the problem with commit order:

```java
 while (true) {
        ConsumerRecords<String, String> records = consumer.poll(100);
        for (ConsumerRecord<String, String> record : records) {
            System.out.printf("topic = %s, partition = %s,
            offset = %d, customer = %s, country = %s\n",
            record.topic(), record.partition(), record.offset(),
            record.key(), record.value());
        }
        consumer.commitAsync(new OffsetCommitCallback() {
            public void onComplete(Map<TopicPartition,
            OffsetAndMetadata> offsets, Exception exception) {
                if (e != null)
                    log.error("Commit failed for offsets {}", offsets, e);
} });
}
```

### Combining Synchronous and Asynchronous Commits
Normally, occasional failures to commit without retrying are not a huge problem because if the problem is temporary, the following commit will be successful. But if we know that this is the last commit before we close the consumer, or before a reba‐ lance, we want to make extra sure that the commit succeeds.
Therefore, a common pattern is to combine commitAsync() with commitSync() just before shutdown. Here is how it works (we will discuss how to commit just before rebalance when we get to the section about rebalance listeners):
```java
 try {
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("topic = %s, partition = %s, offset = %d,
                customer = %s, country = %s\n",
                record.topic(), record.partition(),
                record.offset(), record.key(), record.value());
}
            consumer.commitAsync();
            //While everything is fine, we use commitAsync. It is faster, and if one commit fails,
the next commit will serve as a retry.
        }
    } catch (Exception e) {
        log.error("Unexpected error", e);
    } finally {
        //But if we are closing, there is no “next commit.” We call commitSync(), because it will retry until it succeeds or suffers unrecoverable failure.
        try {
            consumer.commitSync();
        } finally {
            consumer.close();
        }
}
```

### Commit Specified Offset
Committing the latest offset only allows you to commit as often as you finish process‐ ing batches. But what if you want to commit more frequently than that? What if poll() returns a huge batch and you want to commit offsets in the middle of the batch to avoid having to process all those rows again if a rebalance occurs? You can’t just call commitSync() or commitAsync()—this will commit the last offset returned, which you didn’t get to process yet.

Fortunately, the consumer API allows you to call commitSync() and commitAsync() and pass a map of partitions and offsets that you wish to commit. If you are in the middle of processing a batch of records, and the last message you got from partition 3 in topic “customers” has offset 5000, you can call commitSync() to commit offset 5000 for partition 3 in topic “customers.” Since your consumer may be consuming more than a single partition, you will need to track offsets on all of them, which adds complexity to your code.

```java
 private Map<TopicPartition, OffsetAndMetadata> currentOffsets =
        new HashMap<>();
    int count = 0;
    ....
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(100);
        for (ConsumerRecord<String, String> record : records)
        {
            System.out.printf("topic = %s, partition = %s, offset = %d,
            customer = %s, country = %s\n",
            record.topic(), record.partition(), record.offset(),
            record.key(), record.value());
            currentOffsets.put(new TopicPartition(record.topic(),
            record.partition()), new
            OffsetAndMetadata(record.offset()+1, "no metadata"));
            if (count % 1000 == 0)
                consumer.commitAsync(currentOffsets, null);
            count++;
} }
```

### Rebalance Listeners
The consumer API allows you to run your own code when partitions are added or removed from the consumer. You do this by passing a ConsumerRebalanceListener when calling the subscribe() method we discussed previously. ConsumerRebalance Listener has two methods you can implement:

**public void onPartitionsRevoked(Collection<TopicPartition> partitions)**
Called before the rebalancing starts and after the consumer stopped consuming messages. This is where you want to commit offsets, so whoever gets this parti‐ tion next will know where to start.

**public void onPartitionsAssigned(Collection<TopicPartition> partitions)**
Called after partitions have been reassigned to the broker, but before the con‐ sumer starts consuming messages.
```java
private Map<TopicPartition, OffsetAndMetadata> currentOffsets =
      new HashMap<>();
    private class HandleRebalance implements ConsumerRebalanceListener {
        public void onPartitionsAssigned(Collection<TopicPartition>
          partitions) {
        }
        public void onPartitionsRevoked(Collection<TopicPartition>
          partitions) {
            System.out.println("Lost partitions in rebalance.
              Committing current
            offsets:" + currentOffsets);
            consumer.commitSync(currentOffsets);
        }
} try {
     while (true) {
        ConsumerRecords<String, String> records =
          consumer.poll(100);
        for (ConsumerRecord<String, String> record : records)
        {
            System.out.printf("topic = %s, partition = %s, offset = %d,
             customer = %s, country = %s\n",
             record.topic(), record.partition(), record.offset(),
             record.key(), record.value());
             currentOffsets.put(new TopicPartition(record.topic(),
             record.partition()), new
             OffsetAndMetadata(record.offset()+1, "no metadata"));
}
        consumer.commitAsync(currentOffsets, null);
    }
} catch (WakeupException e) {
    // ignore, we're closing
} catch (Exception e) {
    log.error("Unexpected error", e);
} finally {
    try {
        consumer.commitSync(currentOffsets);
    } finally {
        consumer.close();
        System.out.println("Closed consumer and we are done");
    }
}
```

### Consuming Records with Specific Offsets
So far we’ve seen how to use poll() to start consuming messages from the last com‐ mitted offset in each partition and to proceed in processing all messages in sequence. However, sometimes you want to start reading at a different offset.
```java
while (true) {
        ConsumerRecords<String, String> records = consumer.poll(100);
        for (ConsumerRecord<String, String> record : records)
        {
            currentOffsets.put(new TopicPartition(record.topic(),
            record.partition()),
            record.offset());
            processRecord(record);
            storeRecordInDB(record);
            consumer.commitAsync(currentOffsets);
        }
}
```
In this example, we are very paranoid, so we commit offsets after processing each record. However, there is still a chance that our application will crash after the record was stored in the database but before we committed offsets, causing the record to be processed again and the database to contain duplicates.
This could be avoided if there was a way to store both the record and the offset in one atomic action. Either both the record and the offset are committed, or neither of them are committed. As long as the records are written to a database and the offsets to Kafka, this is impossible.

But what if we wrote both the record and the offset to the database, in one transac‐ tion? Then we’ll know that either we are done with the record and the offset is com‐ mitted or we are not and the record will be reprocessed.
Now the only problem is if the record is stored in a database and not in Kafka, how will our consumer know where to start reading when it is assigned a partition? This is exactly what seek() can be used for. When the consumer starts or when new parti‐ tions are assigned, it can look up the offset in the database and seek() to that loca‐ tion.

将offset存到数据库里, 然后通过seek方法修改partition的offset

```java
public class SaveOffsetsOnRebalance implements
      ConsumerRebalanceListener {
        public void onPartitionsRevoked(Collection<TopicPartition>
          partitions) {
                    commitDBTransaction();
}
        public void onPartitionsAssigned(Collection<TopicPartition>
          partitions) {
            for(TopicPartition partition: partitions)
                consumer.seek(partition, getOffsetFromDB(partition));
} }
}
  consumer.subscribe(topics, new SaveOffsetOnRebalance(consumer));
consumer.poll(0);
for (TopicPartition partition: consumer.assignment())
  consumer.seek(partition, getOffsetFromDB(partition));
while (true) {
    ConsumerRecords<String, String> records =
      consumer.poll(100);
    for (ConsumerRecord<String, String> record : records)
    {
        processRecord(record);
        storeRecordInDB(record);
        storeOffsetInDB(record.topic(), record.partition(),
          record.offset());
    }
    commitDBTransaction();
}
```

### But How Do We Exit?
When you decide to exit the poll loop, you will need another thread to call con sumer.wakeup(). If you are running the consumer loop in the main thread, this can be done from ShutdownHook. Note that consumer.wakeup() is the only consumer method that is safe to call from a different thread. Calling wakeup will cause poll() to exit with WakeupException, or if consumer.wakeup() was called while the thread was not waiting on poll, the exception will be thrown on the next iteration when poll() is called. The WakeupException doesn’t need to be handled, but before exiting the thread, you must call consumer.close(). Closing the consumer will commit off‐ sets if needed and will send the group coordinator a message that the consumer is leaving the group. The consumer coordinator will trigger rebalancing immediately and you won’t need to wait for the session to time out before partitions from the con‐ sumer you are closing will be assigned to another consumer in the group.

```java
Runtime.getRuntime().addShutdownHook(new Thread() {
                public void run() {
                    System.out.println("Starting exit...");
                    consumer.wakeup();
                    try {
                        mainThread.join();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
} });
 ... try {
// looping until ctrl-c, the shutdown hook will
   cleanup on exit
while (true) {
    ConsumerRecords<String, String> records =
      movingAvg.consumer.poll(1000);
    System.out.println(System.currentTimeMillis() + "
       --  waiting for data...");
    for (ConsumerRecord<String, String> record :
      records) {
        System.out.printf("offset = %d, key = %s,
          value = %s\n",
          record.offset(), record.key(),
          record.value());
    }
    for (TopicPartition tp: consumer.assignment())
        System.out.println("Committing offset at
          position:" +
          consumer.position(tp));
    movingAvg.consumer.commitSync();
}
} catch (WakeupException e) {
    // ignore for shutdown
} finally {
    consumer.close();
    System.out.println("Closed consumer and we are done");
  } }
```

### Deserializers
```java
 Properties props = new Properties();
    props.put("bootstrap.servers", "broker1:9092,broker2:9092");
    props.put("group.id", "CountryCounter");
    props.put("key.serializer",
       "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.serializer",
       "io.confluent.kafka.serializers.KafkaAvroDeserializer");
    props.put("schema.registry.url", schemaUrl);
    String topic = "customerContacts"
    KafkaConsumer consumer = new
       KafkaConsumer(createConsumerConfig(brokers, groupId, url));
    consumer.subscribe(Collections.singletonList(topic));
    System.out.println("Reading topic:" + topic);
    while (true) {
        ConsumerRecords<String, Customer> records =
          consumer.poll(1000);
        for (ConsumerRecord<String, Customer> record: records) {
            System.out.println("Current customer name is: " +
               record.value().getName());
}
        consumer.commitSync();
    }
```

### Standalone Consumer: Why and How to Use a Consumer Without a Group
So far, we have discussed consumer groups, which are where partitions are assigned automatically to consumers and are rebalanced automatically when consumers are added or removed from the group. Typically, this behavior is just what you want, but in some cases you want something much simpler. Sometimes you know you have a single consumer that always needs to read data from all the partitions in a topic, or from a specific partition in a topic. In this case, there is no reason for groups or reba‐ lances—just assign the consumer-specific topic and/or partitions, consume messages, and commit offsets on occasion.
When you know exactly which partitions the consumer should read, you don’t sub‐ scribe to a topic—instead, you assign yourself a few partitions. A consumer can either subscribe to topics (and be part of a consumer group), or assign itself partitions, but not both at the same time.

```java

    List<PartitionInfo> partitionInfos = null;
    partitionInfos = consumer.partitionsFor("topic");
    //We start by asking the cluster for the partitions available in the topic. If you only plan on consuming a specific partition, you can skip this part.
    if (partitionInfos != null) {
        for (PartitionInfo partition : partitionInfos)
            partitions.add(new TopicPartition(partition.topic(),
              partition.partition()));
        consumer.assign(partitions);
        //Once we know which partitions we want, we call assign() with the list.
        while (true) {
            ConsumerRecords<String, String> records =
              consumer.poll(1000);
            for (ConsumerRecord<String, String> record: records) {
                System.out.printf("topic = %s, partition = %s, offset = %d,
                  customer = %s, country = %s\n",
                  record.topic(), record.partition(), record.offset(),
                  record.key(), record.value());
}
            consumer.commitSync();
        }
}
```
Other than the lack of rebalances and the need to manually find the partitions, every‐ thing else is business as usual. Keep in mind that if someone adds new partitions to the topic, the consumer will not be notified. You will need to handle this by checking consumer.partitionsFor() periodically or simply by bouncing the application whenever partitions are added.

# Kafka Internals
* How Kafka replication works
* How Kafka handles requests from producers and consumers
* How Kafka handles storage such as file format and indexes

## Cluster Membership
Kafka uses Apache Zookeeper to maintain the list of brokers that are currently mem‐ bers of a cluster. Every broker has a unique identifier that is either set in the broker configuration file or automatically generated. Every time a broker process starts, it registers itself with its ID in Zookeeper by creating an ephemeral node. Different Kafka components subscribe to the /brokers/ids path in Zookeeper where brokers are registered so they get notified when brokers are added or removed.

If you try to start another broker with the same ID, you will get an error—the new broker will try to register, but fail because we already have a Zookeeper node for the same broker ID.

When a broker loses connectivity to Zookeeper (usually as a result of the broker stop‐ ping, but this can also happen as a result of network partition or a long garbage- collection pause), the ephemeral node that the broker created when starting will be automatically removed from Zookeeper. Kafka components that are watching the list of brokers will be notified that the broker is gone.

Even though the node representing the broker is gone when the broker is stopped, the broker ID still exists in other data structures. For example, the list of replicas of each topic (see “Replication” on page 97) contains the broker IDs for the replica. This way, if you completely lose a broker and start a brand new broker with the ID of the old one, it will immediately join the cluster in place of the missing broker with the same partitions and topics assigned to it.

## The Controller
The controller is one of the Kafka brokers that, in addition to the usual broker functionality, is responsible for electing partition leaders. The first broker that starts in the cluster becomes the controller by creating an ephemeral node in ZooKeeper called /control ler. When other brokers start, they also try to create this node, but receive a “node already exists” exception, which causes them to “realize” that the controller node already exists and that the cluster already has a controller. The brokers create a Zookeeper watch on the controller node so they get notified of changes to this node. This way, we guarantee that the cluster will only have one controller at a time.

When the controller broker is stopped or loses connectivity to Zookeeper, the ephem‐ eral node will disappear. Other brokers in the cluster will be notified through the Zookeeper watch that the controller is gone and will attempt to create the controller node in Zookeeper themselves. The first node to create the new controller in Zoo‐ keeper is the new controller, while the other nodes will receive a “node already exists” exception and re-create the watch on the new controller node. Each time a controller is elected, it receives a new, higher controller epoch number through a Zookeeper con‐ ditional increment operation. The brokers know the current controller epoch and if they receive a message from a controller with an older number, they know to ignore it.

When the controller notices that a broker left the cluster (by watching the relevant Zookeeper path), it knows that all the partitions that had a leader on that broker will need a new leader. It goes over all the partitions that need a new leader, determines who the new leader should be (simply the next replica in the replica list of that parti‐ tion), and sends a request to all the brokers that contain either the new leaders or the existing followers for those partitions. The request contains information on the new leader and the followers for the partitions. Each new leader knows that it needs to start serving producer and consumer requests from clients while the followers know that they need to start replicating messages from the new leader.

When the controller notices that a broker joined the cluster, it uses the broker ID to check if there are replicas that exist on this broker. If there are, the controller notifies both new and existing brokers of the change, and the replicas on the new broker start replicating messages from the existing leaders.

To summarize, Kafka uses Zookeeper’s ephemeral node feature to elect a controller and to notify the controller when nodes join and leave the cluster. The controller is responsible for electing leaders among the partitions and replicas whenever it notices nodes join and leave the cluster. The controller uses the epoch number to prevent a “split brain” scenario where two nodes believe each is the current controller.

## Replication
As we’ve already discussed, data in Kafka is organized by topics. Each topic is parti‐ tioned, and each partition can have multiple replicas. Those replicas are stored on brokers, and each broker typically stores hundreds or even thousands of replicas belonging to different topics and partitions.

There are two types of replicas:
* Leader replica
Each partition has a single replica designated as the leader. All produce and con‐ sume requests go through the leader, in order to guarantee consistency.

* Follower replica
All replicas for a partition that are not leaders are called followers. Followers don’t serve client requests; their only job is to replicate messages from the leader and stay up-to-date with the most recent messages the leader has. In the event that a leader replica for a partition crashes, one of the follower replicas will be promoted to become the new leader for the partition.

Another task the leader is responsible for is knowing which of the follower replicas is up-to-date with the leader. Followers attempt to stay up-to-date by replicating all the messages from the leader as the messages arrive, but they can fail to stay in sync for various reasons, such as when network congestion slows down replication or when a broker crashes and all replicas on that broker start falling behind until we start the broker and they can start replicating again.

In order to stay in sync with the leader, the replicas send the leader Fetch requests, the exact same type of requests that consumers send in order to consume messages. In response to those requests, the leader sends the messages to the replicas. Those Fetch requests contain the offset of the message that the replica wants to receive next, and will always be in order.

A replica will request message 1, then message 2, and then message 3, and it will not request message 4 before it gets all the previous messages. This means that the leader can know that a replica got all messages up to message 3 when the replica requests message 4. By looking at the last offset requested by each replica, the leader can tell how far behind each replica is. If a replica hasn’t requested a message in more than 10 seconds or if it has requested messages but hasn’t caught up to the most recent mes‐ sage in more than 10 seconds, the replica is considered out of sync. If a replica fails to keep up with the leader, it can no longer become the new leader in the event of failure —after all, it does not contain all the messages.

The inverse of this, replicas that are consistently asking for the latest messages, is called in-sync replicas. Only in-sync replicas are eligible to be elected as partition lead‐ ers in case the existing leader fails.

The amount of time a follower can be inactive or behind before it is considered out of sync is controlled by the replica.lag.time.max.ms configuration parameter. This allowed lag has implications on client behavior and data retention during leader elec‐ tion. We will discuss this in depth in Chapter 6, when we discuss reliability guaran‐ tees.

In addition to the current leader, each partition has a preferred leader—the replica that was the leader when the topic was originally created. It is preferred because when partitions are first created, the leaders are balanced between brokers (we explain the algorithm for distributing replicas and leaders among brokers later in the chapter). As a result, we expect that when the preferred leader is indeed the leader for all parti‐ tions in the cluster, load will be evenly balanced between brokers. By default, Kafka is configured with auto.leader.rebalance.enable=true, which will check if the pre‐ ferred leader replica is not the current leader but is in-sync and trigger leader election to make the preferred leader the current leader.

## Request Processing
For each port the broker listens on, the broker runs an acceptor thread that creates a connection and hands it over to a processor thread for handling. The number of pro‐cessor threads (also called network threads) is configurable. The network threads are responsible for taking requests from client connections, placing them in a request queue, and picking up responses from a response queue and sending them back to cli‐ ents. See Figure 5-1 for a visual of this process.
Once requests are placed on the request queue, IO threads are responsible for picking them up and processing them. The most common types of requests are:

* Produce requests Sent by producers and contain messages the clients write to Kafka brokers.
* Fetch requests Sent by consumers and follower replicas when they read messages from Kafka brokers.

Both produce requests and fetch requests have to be sent to the leader replica of a partition. If a broker receives a produce request for a specific partition and the leader for this partition is on a different broker, the client that sent the produce request will get an error response of “Not a Leader for Partition.” The same error will occur if a fetch request for a specific partition arrives at a broker that does not have the leader for that partition. Kafka’s clients are responsible for sending produce and fetch requests to the broker that contains the leader for the relevant partition for the request.

How do the clients know where to send the requests? Kafka clients use another request type called a metadata request, which includes a list of topics the client is interested in. The server response specifies which partitions exist in the topics, the replicas for each partition, and which replica is the leader. Metadata requests can be sent to any broker because all brokers have a metadata cache that contains this infor‐ mation.

Clients typically cache this information and use it to direct produce and fetch requests to the correct broker for each partition. They also need to occasionally.

refresh this information (refresh intervals are controlled by the meta data.max.age.ms configuration parameter) by sending another metadata request so they know if the topic metadata changed—for example, if a new broker was added or some replicas were moved to a new broker (Figure 5-2). In addition, if a client receives the “Not a Leader” error to one of its requests, it will refresh its metadata before trying to send the request again, since the error indicates that the client is using outdated information and is sending requests to the wrong broker.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_metadata.png" width="700" height="500">
</div>

### Produce Requests
As we saw in Chapter 3, a configuration parameter called acks is the number of brok‐ ers who need to acknowledge receiving the message before it is considered a success‐ ful write. Producers can be configured to consider messages as “written successfully” when the message was accepted by just the leader (acks=1), all in-sync replicas (acks=all), or the moment the message was sent without waiting for the broker to accept it at all (acks=0).
When the broker that contains the lead replica for a partition receives a produce request for this partition, it will start by running a few validations:
* Does the user sending the data have write privileges on the topic?
* Is the number of acks specified in the request valid (only 0, 1, and “all” are allowed)?
* If acks is set to all, are there enough in-sync replicas for safely writing the mes‐ sage? (Brokers can be configured to refuse new messages if the number of in-sync replicas falls below a configurable number; we will discuss this in more detail in Chapter 6, when we discuss Kafka’s durability and reliability guarantees.)

Then it will write the new messages to local disk. On Linux, the messages are written to the filesystem cache and there is no guarantee about when they will be written to disk. Kafka does not wait for the data to get persisted to disk—it relies on replication for message durability.

Once the message is written to the leader of the partition, the broker examines the acks configuration—if acks is set to 0 or 1, the broker will respond immediately; if acks is set to all, the request will be stored in a buffer called purgatory until the leader observes that the follower replicas replicated the message, at which point a response is sent to the client.

### Fetch Requests
Brokers process fetch requests in a way that is very similar to the way produce requests are handled. The client sends a request, asking the broker to send messages from a list of topics, partitions, and offsets—something like “Please send me messages starting at offset 53 in partition 0 of topic Test and messages starting at offset 64 in partition 3 of topic Test.” Clients also specify a limit to how much data the broker can return for each partition. The limit is important because clients need to allocate memory that will hold the response sent back from the broker. Without this limit, brokers could send back replies large enough to cause clients to run out of memory.

As we’ve discussed earlier, the request has to arrive to the leaders of the partitions specified in the request and the client will make the necessary metadata requests to make sure it is routing the fetch requests correctly. When the leader receives the request, it first checks if the request is valid—does this offset even exist for this partic‐ ular partition? If the client is asking for a message that is so old that it got deleted from the partition or an offset that does not exist yet, the broker will respond with an error.

If the offset exists, the broker will read messages from the partition, up to the limit set by the client in the request, and send the messages to the client. Kafka famously uses a zero-copy method to send the messages to the clients—this means that Kafka sends messages from the file (or more likely, the Linux filesystem cache) directly to the net‐ work channel without any intermediate buffers. This is different than most databases where data is stored in a local cache before being sent to clients. This technique removes the overhead of copying bytes and managing buffers in memory, and results in much improved performance.

In addition to setting an upper boundary on the amount of data the broker can return, clients can also set a lower boundary on the amount of data returned. Setting the lower boundary to 10K, for example, is the client’s way of telling the broker “Only return results once you have at least 10K bytes to send me.” This is a great way to reduce CPU and network utilization when clients are reading from topics that are not seeing much traffic. Instead of the clients sending requests to the brokers every few milliseconds asking for data and getting very few or no messages in return, the clients send a request, the broker waits until there is a decent amount of data and returns the data, and only then will the client ask for more (Figure 5-3). The same amount of data is read overall but with much less back and forth and therefore less overhead.

Of course, we wouldn’t want clients to wait forever for the broker to have enough data. After a while, it makes sense to just take the data that exists and process that instead of waiting for more. Therefore, clients can also define a timeout to tell the broker “If you didn’t satisfy the minimum amount of data to send within x milli‐ seconds, just send what you got.”

It is also interesting to note that not all the data that exists on the leader of the parti‐ tion is available for clients to read. Most clients can only read messages that were written to all in-sync replicas (follower replicas, even though they are consumers, are exempt from this—otherwise replication would not work). We already discussed that the leader of the partition knows which messages were replicated to which replica, and until a message was written to all in-sync replicas, it will not be sent to consum‐ ers—attempts to fetch those messages will result in an empty response rather than an error.

The reason for this behavior is that messages not replicated to enough replicas yet are considered “unsafe”—if the leader crashes and another replica takes its place, these messages will no longer exist in Kafka. If we allowed clients to read messages that only exist on the leader, we could see inconsistent behavior. For example, if a con‐ sumer reads a message and the leader crashed and no other broker contained this message, the message is gone. No other consumer will be able to read this message, which can cause inconsistency with the consumer who did read it. Instead, we wait until all the in-sync replicas get the message and only then allow consumers to read it (Figure 5-4). This behavior also means that if replication between brokers is slow for some reason, it will take longer for new messages to arrive to consumers (since we wait for the messages to replicate first). This delay is limited to replica.lag.time.max.ms—the amount of time a replica can be delayed in replicat‐ ing new messages while still being considered in-sync.

## Physical Storage
First, we want to look at how data is allocated to the brokers in the cluster and the directories in the broker. Then we will look at how the broker manages the files—especially how the retention guarantees are handled. We will then dive inside the files and look at the file and index formats. Lastly we will look at Log Compaction, an advanced feature that allows turning Kafka into a long-term data store, and describe how it works.

### Partition Allocation
When you create a topic, Kafka first decides how to allocate the partitions between brokers. Suppose you have 6 brokers and you decide to create a topic with 10 parti‐ tions and a replication factor of 3. Kafka now has 30 partition replicas to allocate to 6 brokers. When doing the allocations, the goals are:
* To spread replicas evenly among brokers—in our example, to make sure we allo‐ cate 5 replicas per broker.
* To make sure that for each partition, each replica is on a different broker. If parti‐ tion 0 has the leader on broker 2, we can place the followers on brokers 3 and 4, but not on 2 and not both on 3.
* If the brokers have rack information (available in Kafka release 0.10.0 and higher), then assign the replicas for each partition to different racks if possible. This ensures that an event that causes downtime for an entire rack does not cause complete unavailability for partitions.

To do this, we start with a random broker (let’s say, 4) and start assigning partitions to each broker in round-robin manner to determine the location for the leaders. So par‐ tition leader 0 will be on broker 4, partition 1 leader will be on broker 5, partition 2 will be on broker 0 (because we only have 6 brokers), and so on. Then, for each parti‐ tion, we place the replicas at increasing offsets from the leader. If the leader for parti‐ tion 0 is on broker 4, the first follower will be on broker 5 and the second on broker 0. The leader for partition 1 is on broker 5, so the first replica is on broker 0 and the second on broker 1.

When rack awareness is taken into account, instead of picking brokers in numerical order, we prepare a rack-alternating broker list. Suppose that we know that brokers 0, 1, and 2 are on the same rack, and brokers 3, 4, and 5 are on a separate rack. Instead of picking brokers in the order of 0 to 5, we order them as 0, 3, 1, 4, 2, 5—each broker is followed by a broker from a different rack (Figure 5-5). In this case, if the leader for partition 0 is on broker 4, the first replica will be on broker 2, which is on a com‐ pletely different rack. This is great, because if the first rack goes offline, we know that we still have a surviving replica and therefore the partition is still available. This will be true for all our replicas, so we have guaranteed availability in the case of rack fail‐ ure.

Once we choose the correct brokers for each partition and replica, it is time to decide which directory to use for the new partitions. We do this independently for each par‐ tition, and the rule is very simple: we count the number of partitions on each direc‐ tory and add the new partition to the directory with the fewest partitions. This means that if you add a new disk, all the new partitions will be created on that disk. This is because, until things balance out, the new disk will always have the fewest partitions.

### File Management
Retention is an important concept in Kafka—Kafka does not keep data forever, nor does it wait for all consumers to read a message before deleting it. Instead, the Kafka administrator configures a retention period for each topic—either the amount of time to store messages before deleting them or how much data to store before older mes‐ sages are purged.

Because finding the messages that need purging in a large file and then deleting a portion of the file is both time-consuming and error-prone, we instead split each par‐ tition into segments. By default, each segment contains either 1 GB of data or a week of data, whichever is smaller. As a Kafka broker is writing to a partition, if the seg‐ ment limit is reached, we close the file and start a new one.

The segment we are currently writing to is called an active segment. The active seg‐ ment is never deleted, so if you set log retention to only store a day of data but each segment contains five days of data, you will really keep data for five days because we can’t delete the data before the segment is closed. If you choose to store data for a week and roll a new segment every day, you will see that every day we will roll a new segment while deleting the oldest segment—so most of the time the partition will have seven segments.

### File Format
Each segment is stored in a single data file. Inside the file, we store Kafka messages and their offsets. The format of the data on the disk is identical to the format of the messages that we send from the producer to the broker and later from the broker to the consumers. Using the same message format on disk and over the wire is what allows Kafka to use zero-copy optimization when sending messages to consumers and also avoid decompressing and recompressing messages that the producer already compressed.

Each message contains—in addition to its key, value, and offset—things like the mes‐ sage size, checksum code that allows us to detect corruption, magic byte that indicates the version of the message format, compression codec (Snappy, GZip, or LZ4), and a timestamp (added in release 0.10.0). The timestamp is given either by the producer when the message was sent or by the broker when the message arrived—depending on configuration.

If the producer is sending compressed messages, all the messages in a single producer batch are compressed together and sent as the “value” of a “wrapper message” (Figure 5-6). So the broker receives a single message, which it sends to the consumer. But when the consumer decompresses the message value, it will see all the messages that were contained in the batch, with their own timestamps and offsets.

This means that if you are using compression on the producer (recommended!), sending larger batches means better compression both over the network and on the broker disks. This also means that if we decide to change the message format that consumers use (e.g., add a timestamp to the message), both the wire protocol and the on-disk format need to change, and Kafka brokers need to know how to handle cases in which files contain messages of two formats due to upgrades.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_fileformat.png" width="700" height="400">
</div>

### Indexes
Kafka allows consumers to start fetching messages from any available offset. This means that if a consumer asks for 1 MB messages starting at offset 100, the broker must be able to quickly locate the message for offset 100 (which can be in any of the segments for the partition) and start reading the messages from that offset on. In order to help brokers quickly locate the message for a given offset, Kafka maintains an index for each partition. The index maps offsets to segment files and positions within the file.

Indexes are also broken into segments, so we can delete old index entries when the messages are purged. Kafka does not attempt to maintain checksums of the index. If the index becomes corrupted, it will get regenerated from the matching log segment simply by rereading the messages and recording the offsets and locations. It is also completely safe for an administrator to delete index segments if needed—they will be regenerated automatically.

### Compaction
Normally, Kafka will store messages for a set amount of time and purge messages older than the retention period. However, imagine a case where you use Kafka to store shipping addresses for your customers. In that case, it makes more sense to store the last address for each customer rather than data for just the last week or year. This way, you don’t have to worry about old addresses and you still retain the address for customers who haven’t moved in a while. Another use case can be an application that uses Kafka to store its current state. Every time the state changes, the application writes the new state into Kafka. When recovering from a crash, the application reads those messages from Kafka to recover its latest state. In this case, it only cares about the latest state before the crash, not all the changes that occurred while it was run‐ ning.

Kafka supports such use cases by allowing the retention policy on a topic to be delete, which deletes events older than retention time, to compact, which only stores the most recent value for each key in the topic. Obviously, setting the policy to compact only makes sense on topics for which applications produce events that contain both a key and a value. If the topic contains null keys, compaction will fail.
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_clean.png" width="700" height="300">
</div>

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_compaction.png" width="700" height="300">
</div>

In order to delete a key from the system completely, not even saving the last message, the application must produce a message that contains that key and a null value. When the cleaner thread finds such a message, it will first do a normal compaction and retain only the message with the null value. It will keep this special message (known as a tombstone) around for a configurable amount of time. During this time, consum‐ ers will be able to see this message and know that the value is deleted. 

### When Are Topics Compacted?
In the same way that the delete policy never deletes the current active segments, the compact policy never compacts the current segment. Messages are eligble for compac‐ tion only on inactive segments.
In version 0.10.0 and older, Kafka will start compacting when 50% of the topic con‐ tains dirty records. The goal is not to compact too often (since compaction can impact the read/write performance on a topic), but also not leave too many dirty records around (since they consume disk space).

# Reliable Data Delivery

## Reliability Guarantees
* Kafka provides order guarantee of messages in a partition. If message B was writ‐ ten after message A, using the same producer in the same partition, then Kafka guarantees that the offset of message B will be higher than message A, and that consumers will read message B after message A.
* Produced messages are considered “committed” when they were written to the partition on all its in-sync replicas (but not necessarily flushed to disk). Produc‐ ers can choose to receive acknowledgments of sent messages when the message was fully committed, when it was written to the leader, or when it was sent over the network.
* Messages that are committed will not be lost as long as at least one replica remains alive.
* Consumers can only read messages that are committed.

## Replication
Each Kafka topic is broken down into partitions, which are the basic data building blocks. A partition is stored on a single disk. Kafka guarantees order of events within a partition and a partition can be either online (available) or offline (unavailable). Each partition can have multiple replicas, one of which is a designated leader. All events are produced to and consumed from the leader replica. Other replicas just need to stay in sync with the leader and replicate all the recent events on time. If the leader becomes unavailable, one of the in-sync replicas becomes the new leader.

A replica is considered in-sync if it is the leader for a partition, or if it is a follower that:
* Has an active session with Zookeeper—meaning, it sent a heartbeat to Zookeeper in the last 6 seconds (configurable).
* Fetched messages from the leader in the last 10 seconds (configurable).
* Fetched the most recent messages from the leader in the last 10 seconds. That is, it isn’t enough that the follower is still getting messages from the leader; it must have almost no lag.

An in-sync replica that is slightly behind can slow down producers and consumers— since they wait for all the in-sync replicas to get the message before it is committed. Once a replica falls out of sync, we no longer wait for it to get messages. It is still behind, but now there is no performance impact. The catch is that with fewer in-sync replicas, the effective replication factor of the partition is lower and therefore there is a higher risk for downtime or data loss.
## Broker Configuration
There are three configuration parameters in the broker that change Kafka’s behavior regarding reliable message storage. Like many broker configuration variables, these can apply at the broker level, controlling configuration for all topics in the system, and at the topic level, controlling behavior for a specific topic.

Being able to control reliability trade-offs at the topic level means that the same Kafka cluster can be used to host reliable and nonreliable topics. For example, at a bank, the administrator will probably want to set very reliable defaults for the entire cluster but make an exception to the topic that stores customer complaints where some data loss is acceptable.

### Replication Factor
If you are totally OK with a specific topic being unavailable when a single broker is restarted (which is part of the normal operations of a cluster), then a replication fac‐ tor of 1 may be enough. Don’t forget to make sure your management and users are also OK with this trade-off—you are saving on disks or servers, but losing high avail‐ ability. A replication factor of 2 means you can lose one broker and still be OK, which sounds like enough, but keep in mind that losing one broker can sometimes (mostly on older versions of Kafka) send the cluster into an unstable state, forcing you to restart another broker—the Kafka Controller. This means that with a replication fac‐ tor of 2, you may be forced to go into unavailability in order to recover from an operational issue. This can be a tough choice.

For those reasons, we recommend a replication factor of 3 for any topic where availa‐ bility is an issue. In rare cases, this is considered not safe enough—we’ve seen banks run critical topics with five replicas, just in case.

Placement of replicas is also very important. By default, Kafka will make sure each replica for a partition is on a separate broker. However, in some cases, this is not safe enough. If all replicas for a partition are placed on brokers that are on the same rack and the top-of-rack switch misbehaves, you will lose availability of the partition regardless of the replication factor. To protect against rack-level misfortune, we rec‐ ommend placing brokers in multiple racks and using the broker.rack broker config‐ uration parameter to configure the rack name for each broker. If rack names are configured, Kafka will make sure replicas for a partition are spread across multiple racks in order to guarantee even higher availability. In Chapter 5 we provided details on how Kafka places replicas on brokers and racks, if you are interested in under‐ standing more.

### Unclean Leader Election
This configuration is only available at the broker (and in practice, cluster-wide) level. The parameter name is unclean.leader.election.enable and by default it is set to true.

As explained earlier, when the leader for a partition is no longer available, one of the in-sync replicas will be chosen as the new leader. This leader election is “clean” in the sense that it guarantees no loss of committed data—by definition, committed data exists on all in-sync replicas.

But what do we do when no in-sync replica exists except for the leader that just became unavailable?

This situation can happen in one of two scenarios:
* The partition had three replicas, and the two followers became unavailable (let’s say two brokers crashed). In this situation, as producers continue writing to the leader, all the messages are acknowledged and committed (since the leader is the one and only in-sync replica). Now let’s say that the leader becomes unavailable (oops, another broker crash). In this scenario, if one of the out-of-sync followers starts first, we have an out-of-sync replica as the only available replica for the partition.
* The partition had three replicas and, due to network issues, the two followers fell behind so that even though they are up and replicating, they are no longer in sync. The leader keeps accepting messages as the only in-sync replica. Now if the leader becomes unavailable, the two available replicas are no longer in-sync.

In summary, if we allow out-of-sync replicas to become leaders, we risk data loss and data inconsistencies. If we don’t allow them to become leaders, we face lower availa‐ bility as we must wait for the original leader to become available before the partition is back online. 

Setting unclean.leader.election.enable to true means we allow out-of-sync repli‐ cas to become leaders (knowns as unclean election), knowing that we will lose mes‐ sages when this occurs. If we set it to false, we choose to wait for the original leader to come back online, resulting in lower availability. We typically see unclean leader elec‐ tion disabled (configuration set to false) in systems where data quality and consis‐ tency are critical—banking systems are a good example. In systems where availability is more important, such as real-time clickstream analysis, unclean leader election is often enabled.

### Minimum In-Sync Replicas
Both the topic and the broker-level configuration are called min.insync.replicas.

As we’ve seen, there are cases where even though we configured a topic to have three replicas, we may be left with a single in-sync replica. If this replica becomes unavail‐ able, we may have to choose between availability and consistency. This is never an easy choice. Note that part of the problem is that, per Kafka reliability guarantees, data is considered committed when it is written to all in-sync replicas, even when all means just one replica and the data could be lost if that replica is unavailable.

If you would like to be sure that committed data is written to more than one replica, you need to set the minimum number of in-sync replicas to a higher value. If a topic has three replicas and you set min.insync.replicas to 2, then you can only write to a partition in the topic if at least two out of the three replicas are in-sync.

When all three replicas are in-sync, everything proceeds normally. This is also true if one of the replicas becomes unavailable. However, if two out of three replicas are not available, the brokers will no longer accept produce requests. Instead, producers that attempt to send data will receive NotEnoughReplicasException. Consumers can con‐ tinue reading existing data. In effect, with this configuation, a single in-sync replica becomes read-only. This prevents the undesirable situation where data is produced and consumed, only to disappear when unclean election occurs. In order to recover from this read-only situation, we must make one of the two unavailable partitions available again (maybe restart the broker) and wait for it to catch up and get in-sync.

## Using Producers in a Reliable System
Even if we configure the brokers in the most reliable configuration possible, the sys‐ tem as a whole can still accidentally lose data if we don’t configure the producers to be reliable as well.
Here are two example scenarios to demonstrate this:

* We configured the brokers with three replicas, and unclean leader election is dis‐ abled. So we should never lose a single message that was committed to the Kafka cluster. However, we configured the producer to send messages with acks=1. We send a message from the producer and it was written to the leader, but not yet to the in-sync replicas. The leader sent back a response to the producer saying “Message was written successfully” and immediately crashes before the data was replicated to the other replicas. The other replicas are still considered in-sync (remember that it takes a while before we declare a replica out of sync) and one of them will become the leader. Since the message was not written to the replicas, it will be lost. But the producing application thinks it was written successfully. The system is consistent because no consumer saw the message (it was never committed because the replicas never got it), but from the producer perspective, a message was lost.

* We configured the brokers with three replicas, and unclean leader election is dis‐ abled. We learned from our mistakes and started producing messages with acks=all. Suppose that we are attempting to write a message to Kafka, but the leader for the partition we are writing to just crashed and a new one is still get‐ ting elected. Kafka will respond with “Leader not Available.” At this point, if the producer doesn’t handle the error correctly and doesn’t retry until the write is successful, the message may be lost. Once again, this is not a broker reliability issue because the broker never got the message; and it is not a consistency issue because the consumers never got the message either. But if producers don’t han‐ dle errors correctly, they may cause message loss.

So how do we avoid these tragic results? As the examples show, there are two impor‐ tant things that everyone who writes applications that produce to Kafka must pay attention to:
* Use the correct acks configuration to match reliability requirements
* Handle errors correctly both in configuration and in code
  
### Configuring Producer Retries
The producer can handle retriable errors that are returned by the broker for you. When the producer sends messages to a broker, the broker can return either a success or an error code. Those error codes belong to two categories—errors that can be resolved after retrying and errors that won’t be resolved. For example, if the broker returns the error code LEADER_NOT_AVAILABLE, the producer can try sending the error again—maybe a new broker was elected and the second attempt will succeed. This means that LEADER_NOT_AVAILABLE is a retriable error. On the other hand, if a broker returns an INVALID_CONFIG exception, trying the same message again will not change the configuration. This is an example of a nonretriable error.

通常retry要设置一个up limit, drop掉并不是一个很好的做法

kafka保证message被store至少一次, 但不保证exactly once, 不过会有unique identifier 来检测重复


### ffective strategy to avoid duplicate messages in kafka consumer
kafka本身没有解决这个问题的能力, 总会产生duplicate, 因此需要在consumer和producer我们自己这一端解决才行

The short answer is, no.

What you're looking for is exactly-once processing. While it may often seem feasible, it should never be relied upon because there are always caveats.

Even in order to attempt to prevent duplicates you would need to use the simple consumer. How this approach works is for each consumer, when a message is consumed from some partition, write the partition and offset of the consumed message to disk. When the consumer restarts after a failure, read the last consumed offset for each partition from disk.

But even with this pattern the consumer can't guarantee it won't reprocess a message after a failure. What if the consumer consumes a message and then fails before the offset is flushed to disk? If you write to disk before you process the message, what if you write the offset and then fail before actually processing the message? This same problem would exist even if you were to commit offsets to ZooKeeper after every message.

Exactly once semantics has two parts: avoiding duplication during data production and avoiding duplicates during data consumption.

There are two approaches to getting exactly once semantics during data production:

* Use a single-writer per partition and every time you get a network error check the last message in that partition to see if your last write succeeded
* Include a primary key (UUID or something) in the message and deduplicate on the consumer.

**Other applications make the messages idempotent—meaning that even if the same message is sent twice, it has no negative impact on correctness.**

## Using Consumers in a Reliable System
The main way consumers can lose messages is when committing offsets for events they’ve read but didn’t completely process yet. This way, when another consumer picks up the work, it will skip those events and they will never get processed. This is why paying careful attention to when and how offsets get committed is critical.

### Important Consumer Configuration Properties for Reliable Processing
There are four consumer configuration properties that are important to understand in order to configure your consumer for a desired reliability behavior.

The first is group.id, as explained in great detail in Chapter 4. The basic idea is that if two consumers have the same group ID and subscribe to the same topic, each will be assigned a subset of the partitions in the topic and will therefore only read a subset of the messages individually (but all the messages will be read by the group as a whole). If you need a consumer to see, on its own, every single message in the topics it is sub‐ scribed to—it will need a unique group.id.

The second relevant configuration is auto.offset.reset. This parameter controls what the consumer will do when no offsets were committed (e.g., when the consumer first starts) or when the consumer asks for offsets that don’t exist in the broker (Chap‐ ter 4 explains how this can happen). There are only two options here. If you choose earliest, the consumer will start from the beginning of the partition whenever it doesn’t have a valid offset. This can lead to the consumer processing a lot of messages twice, but it guarantees to minimize data loss. If you choose latest, the consumer will start at the end of the partition. This minimizes duplicate processing by the con‐ sumer but almost certainly leads to some messages getting missed by the consumer.

The third relevant configuration is enable.auto.commit. This is a big decision: are you going to let the consumer commit offsets for you based on schedule, or are you planning on committing offsets manually in your code? The main benefit of auto‐ matic offset commits is that it’s one less thing to worry about when implementing your consumers. If you do all the processing of consumed records within the con‐ sumer poll loop, then the automatic offset commit guarantees you will never commit an offset that you didn’t process. (If you are not sure what the consumer poll loop is, refer back to Chapter 4.) The main drawbacks of automatic offset commits is that you have no control over the number of duplicate records you may need to process (because your consumer stopped after processing some records but before the auto‐ mated commit kicked in). If you do anything fancy like pass records to another thread to process in the background, the automatic commit may commit offsets for records the consumer has read but perhaps did not process yet.

The fourth relevant configuration is tied to the third, and is auto.com mit.interval.ms. If you choose to commit offsets automatically, this configuration lets you configure how frequently they will be committed. The default is every five seconds. In general, committing more frequently adds some overhead but reduces the number of duplicates that can occur when a consumer stops.

### Explicitly Committing Offsets in Consumers
**Always commit offsets after events were processed**
If you do all the processing within the poll loop and don’t maintain state between poll loops (e.g., for aggregation), this should be easy. You can use the auto-commit config‐ uration or commit events at the end of the poll loop.

**Commit frequency is a trade-off between performance and number of duplicates in the event of a crash**

Even in the simplest case where you do all the processing within the poll loop and don’t maintain state between poll loops, you can choose to commit multiple times within a loop (perhaps even after every event) or choose to only commit every several loops. Committing has some performance overhead (similar to produce with acks=all), so it all depends on the trade-offs that work for you.

**Make sure you know exactly what offsets you are committing**

A common pitfall when committing in the middle of the poll loop is accidentally committing the last offset read when polling and not the last offset processed. Remember that it is critical to always commit offsets for messages after they were processed—committing offsets for messages read but not processed can lead to the consumer missing messages. Chapter 4 has examples that show how to do just that.

**Rebalances**

When designing your application, remember that consumer rebalances will happen and you need to handle them properly. Chapter 4 contains a few examples, but the bigger picture is that this usually involves committing offsets before partitions are revoked and cleaning any state you maintain when you are assigned new partitions.

**Consumers may need to retry**

In some cases, after calling poll and processing records, some records are not fully processed and will need to be processed later. For example, you may try to write records from Kafka to a database, but find that the database is not available at that moment and you may wish to retry later. Note that unlike traditional pub/sub mes‐ saging systems, you commit offsets and not ack individual messages. This means that if you failed to process record #30 and succeeded in processing record #31, you should not commit record #31—this would result in committing all the records up to #31 including #30, which is usually not what you want. Instead, try following one of the following two patterns.

One option, when you encounter a retriable error, is to commit the last record you processed successfully. Then store the records that still need to be processed in a buffer (so the next poll won’t override them) and keep trying to process the records. You may need to keep polling while trying to process all the records (refer to Chap‐ ter 4 for an explanation). You can use the consumer pause() method to ensure that additional polls won’t return additional data to make retrying easier.

A second option is, when encountering a retriable error, to write it to a separate topic and continue. A separate consumer group can be used to handle retries from the retry topic, or one consumer can subscribe to both the main topic and to the retry topic, but pause the retry topic between retries. This pattern is similar to the dead- letter-queue system used in many messaging systems.

**Consumers may need to maintain state**

In some applications, you need to maintain state across multiple calls to poll. For example, if you want to calculate moving average, you’ll want to update the average after every time you poll Kafka for new events. If your process is restarted, you will need to not just start consuming from the last offset, but you’ll also need to recover the matching moving average. One way to do this is to write the latest accumulated value to a “results” topic at the same time you are committing the offset. This means that when a thread is starting up, it can pick up the latest accumulated value when it starts and pick up right where it left off. However, this doesn’t completely solve the problem, as Kafka does not offer transactions yet. You could crash after you wrote the latest result and before you committed offsets, or vice versa. In general, this is a rather complex problem to solve, and rather than solving it on your own, we recommend looking at a library like Kafka Streams, which provides high level DSL-like APIs for aggregation, joins, windows, and other complex analytics.

**Handling long processing times**

Sometimes processing records takes a long time. Maybe you are interacting with a service that can block or doing a very complex calculation, for example. Remember that in some versions of the Kafka consumer, you can’t stop polling for more than a few seconds (see Chapter 4 for details). Even if you don’t want to process additional records, you must continue polling so the client can send heartbeats to the broker. A common pattern in these cases is to hand off the data to a thread-pool when possible with multiple threads to speed things up a bit by processing in parallel. After handing off the records to the worker threads, you can pause the consumer and keep polling without actually fetching additional data until the worker threads finish. Once they are done, you can resume the consumer. Because the consumer never stops polling, the heartbeat will be sent as planned and rebalancing will not be triggered.

**Exactly-once delivery**
Some applications require not just at-least-once semantics (meaning no data loss), but also exactly-once semantics. While Kafka does not provide full exactly-once sup‐ port at this time, consumers have few tricks available that allow them to guarantee that each message in Kafka will be written to an external system exactly once (note that this doesn’t handle duplications that may have occurred while the data was pro‐ duced into Kafka).

The easiest and probably most common way to do exactly-once is by writing results to a system that has some support for unique keys. This includes all key-value stores, all relational databases, Elasticsearch, and probably many more data stores. When writing results to a system like a relational database or Elastic search, either the record itself contains a unique key (this is fairly common), or you can create a unique key using the topic, partition, and offset combination, which uniquely identifies a Kafka record. If you write the record as a value with a unique key, and later you acci‐ dentally consume the same record again, you will just write the exact same key and value. The data store will override the existing one, and you will get the same result that you would without the accidental duplicate. This pattern is called idempotent writes and is very common and useful.

Another option is available when writing to a system that has transactions. Relational databases are the easiest example, but HDFS has atomic renames that are often used for the same purpose. The idea is to write the records and their offsets in the same transaction so they will be in-sync. When starting up, retrieve the offsets of the latest records written to the external store and then use consumer.seek() to start consuming again from those offsets. Chapter 4 contains an example of how this can be done.

# Building Data Pipelines
The main value Kafka provides to data pipelines is its ability to serve as a very large, reliable buffer between various stages in the pipeline, effectively decoupling produc‐ ers and consumers of data within the pipeline. This decoupling, combined with relia‐ bility security and efficiency, makes Kafka a good fit for most data pipelines.

## Considerations When Building Data Pipelines

### Timeliness
Some systems expect their data to arrive in large bulks once a day; others expect the data to arrive a few milliseconds after it is generated. A useful way to look at Kafka in this context is that it acts as a giant buffer that decou‐ ples the time-sensitivity requirements between producers and consumers. Producers can write events in real-time while consumers process batches of events, or vice versa. This also makes it trivial to apply back-pressure—Kafka itself applies back-pressure on producers (by delaying acks when needed) since consumption rate is driven entirely by the consumers.

### Reliability
We want to avoid single points of failure and allow for fast and automatic recovery from all sorts of failure events. Data pipelines are often the way data arrives to busi‐ ness critical systems; failure for more than a few seconds can be hugely disruptive, especially when the timeliness requirement is closer to the few-milliseconds end of the spectrum. Another important consideration for reliability is delivery guarantees —some systems can afford to lose data, but most of the time there is a requirement for at-least-once delivery, which means every event from the source system will reach its destination, but sometimes retries will cause duplicates. Often, there is even a requirement for exactly-once delivery—every event from the source system will reach the destination with no possibility for loss or duplication.

We discussed Kafka’s availability and reliability guarantees in depth in Chapter 6. As we discussed, Kafka can provide at-least-once on its own, and exactly-once when combined with an external data store that has a transactional model or unique keys. Since many of the end points are data stores that provide the right semantics for exactly-once delivery, a Kafka-based pipeline can often be implemented as exactly- once. It is worth highlighting that Kafka’s Connect APIs make it easier for connectors to build an end-to-end exactly-once pipeline by providing APIs for integrating with the external systems when handling offsets. Indeed, many of the available open source connectors support exactly-once delivery.

### High and Varying Throughput
The data pipelines we are building should be able to scale to very high throughputs as is often required in modern data systems. Even more importantly, they should be able to adapt if throughput suddenly increases.

With Kafka acting as a buffer between producers and consumers, we no longer need to couple consumer throughput to the producer throughput. We no longer need to implement a complex back-pressure mechanism because if producer throughput exceeds that of the consumer, data will accumulate in Kafka until the consumer can catch up. Kafka’s ability to scale by adding consumers or producers independently allows us to scale either side of the pipeline dynamically and independently to match the changing requirements.

### Data Formats
One of the most important considerations in a data pipeline is reconciling different data formats and data types. The data types supported vary among different databases and other storage systems. You may be loading XMLs and relational data into Kafka, using Avro within Kafka, and then need to convert data to JSON when writing it to Elasticsearch, to Parquet when writing to HDFS, and to CSV when writing to S3.

Kafka itself and the Connect APIs are completely agnostic when it comes to data for‐ mats. As we’ve seen in previous chapters, producers and consumers can use any seri‐ alizer to represent data in any format that works for you. Kafka Connect has its own in-memory objects that include data types and schemas, but as we’ll soon discuss, it allows for pluggable converters to allow storing these records in any format. This means that no matter which data format you use for Kafka, it does not restrict your choice of connectors.

### Transformations
Transformations are more controversial than other requirements. There are generally two schools of building data pipelines: ETL and ELT. ETL, which stands for Extract- Transform-Load, means the data pipeline is responsible for making modifications to the data as it passes through. It has the perceived benefit of saving time and storage because you don’t need to store the data, modify it, and store it again. 

ELT stands for Extract-Load-Transform and means the data pipeline does only mini‐ mal transformation (mostly around data type conversion), with the goal of making sure the data that arrives at the target is as similar as possible to the source data.

### Security
* Can we make sure the data going through the pipe is encrypted? This is mainly a concern for data pipelines that cross datacenter boundaries.
* Who is allowed to make modifications to the pipelines?
* If the data pipeline needs to read or write from access-controlled locations, can it authenticate properly?

## Kafka Connect
Kafka Connect ships with Apache Kafka, so there is no need to install it separately. For production use, especially if you are planning to use Connect to move large amounts of data or run many connectors, you should run Connect on separate servers. In this case, install Apache Kafka on all the machines, and simply start the brokers on some servers and start Connect on other servers.
Starting a Connect worker is very similar to starting a broker—you call the start script with a properties file:

```bin/connect-distributed.sh config/connect-distributed.properties```

There are a few key configurations for Connect workers:
* bootstrap.servers:: A list of Kafka brokers that Connect will work with. Connectors will pipe their data either to or from those brokers. You don’t need to specify every broker in the cluster, but it’s recommended to specify at least three.
* group.id:: All workers with the same group ID are part of the same Connect cluster. A connector started on the cluster will run on any worker and so will its tasks.
* key.converter and value.converter:: Connect can handle multiple data for‐ mats stored in Kafka. The two configurations set the converter for the key and value part of the message that will be stored in Kafka. The default is JSON format using the JSONConverter included in Apache Kafka. These configurations can also be set to AvroConverter, which is part of the Confluent Schema Registry.

## A Deeper Look at Connect
The best way to understand workers is to realize that connectors and tasks are responsible for the “moving data” part of data integration, while the workers are responsible for the REST API, configuration management, reliability, high availabil‐ ity, scaling, and load balancing.

This separation of concerns is the main benefit of using Connect APIs versus the clas‐ sic consumer/producer APIs. Experienced developers know that writing code that reads data from Kafka and inserts it into a database takes maybe a day or two, but if you need to handle configuration, errors, REST APIs, monitoring, deployment, scal‐ ing up and down, and handling failures, it can take a few months to get right. If you implement data copying with a connector, your connector plugs into workers that handle a bunch of complicated operational issues that you don’t need to worry about.

# Cross-Cluster Data Mirroring
In this chapter we will discuss cross-cluster mirroring of all or part of the data. We’ll start by discussing some of the common use cases for cross-cluster mirroring. Then we’ll show a few architectures that are used to implement these use cases and discuss the pros and cons of each architecture pattern. We’ll then discuss MirrorMaker itself and how to use it. 

## Use Cases of Cross-Cluster Mirroring
The following is a list of examples of when cross-cluster mirroring would be used.

**Regional and central clusters**

In some cases, the company has one or more datacenters in different geographi‐ cal regions, cities, or continents. Each datacenter has its own Kafka cluster. Some applications can work just by communicating with the local cluster, but some applications require data from multiple datacenters (otherwise, you wouldn’t be looking at cross data-center replication solutions). There are many cases when this is a requirement, but the classic example is a company that modifies prices based on supply and demand. This company can have a datacenter in each city in which it has a presence, collects information about local supply and demand, and adjusts prices accordingly. All this information will then be mirrored to a central cluster where business analysts can run company-wide reports on its revenue.

**Redundancy (DR)**

The applications run on just one Kafka cluster and don’t need data from other locations, but you are concerned about the possibility of the entire cluster becoming unavailable for some reason. You’d like to have a second Kafka cluster with all the data that exists in the first cluster, so in case of emergency you can direct your applications to the second cluster and continue as usual.

**Cloud migrations**

Many companies these days run their business in both an on-premise datacenter and a cloud provider. Often, applications run on multiple regions of the cloud provider, for redundancy, and sometimes multiple cloud providers are used. In these cases, there is often at least one Kafka cluster in each on-premise datacenter and each cloud region. Those Kafka clusters are used by applications in each datacenter and region to transfer data efficiently between the datacenters. For example, if a new application is deployed in the cloud but requires some data that is updated by applications running in the on-premise datacenter and stored in an on-premise database, you can use Kafka Connect to capture database changes to the local Kafka cluster and then mirror these changes to the cloud Kafka cluster where the new application can use them. This helps control the costs of cross- datacenter traffic as well as improve governance and security of the traffic.

We’ll talk more about tuning Kafka for cross-datacenter communication, but the fol‐ lowing principles will guide most of the architectures we’ll discuss next:

* No less than one cluster per datacenter
* Replicate each event exactly once (barring retries due to errors) between each pair of datacenters
* When possible, consume from a remote datacenter rather than produce to a remote datacenter

# Stream Processing

## What Is Stream Processing?
Let’s start at the beginning: What is a data stream (also called an event stream or streaming data)? First and foremost, a data stream is an abstraction representing an unbounded dataset. Unbounded means infinite and ever growing. The dataset is unbounded because over time, new records keep arriving. This definition is used by Google, Amazon, and pretty much everyone else.

Note that this simple model (a stream of events) can be used to represent pretty much every business activity we care to analyze. We can look at a stream of credit card transactions, stock trades, package deliveries, network events going through a switch, events reported by sensors in manufacturing equipment, emails sent, moves in a game, etc. The list of examples is endless because pretty much everything can be seen as a sequence of events.

There are few other attributes of event streams model, in addition to their unboun‐ ded nature:

**Event streams are ordered**

There is an inherent notion of which events occur before or after other events. This is clearest when looking at financial events. A sequence in which I first put money in my account and later spend the money is very different from a sequence at which I first spend the money and later cover my debt by depositing money back. The latter will incur overdraft charges while the former will not. Note that this is one of the differences between an event stream and a database table—records in a table are always considered unordered and the “order by” clause of SQL is not part of the relational model; it was added to assist in reporting.

**Immutable data records**

Events, once occured, can never be modified. A financial transaction that is can‐ celled does not disapear. Instead, an additional event is written to the stream, recording a cancellation of previous transaction. When a customer returns mer‐ chandise to a shop, we don’t delete the fact that the merchandise was sold to him earlier, rather we record the return as an additional event. This is another differ‐ ence between a data stream and a database table—we can delete or update records in a table, but those are all additional transactions that occur in the data‐ base, and as such can be recorded in a stream of events that records all transac‐ tions. If you are familiar with binlogs, WALs, or redo logs in databases you can see that if we insert a record into a table and later delete it, the table will no longer contain the record, but the redo log will contain two transactions—the insert and the delete.

**Event streams are replayable**

This is a desirable property. While it is easy to imagine nonreplayable streams (TCP packets streaming through a socket are generally nonreplayable), for most business applications, it is critical to be able to replay a raw stream of events that occured months (and sometimes years) earlier. This is required in order to cor‐ rect errors, try new methods of analysis, or perform audits. This is the reason we believe Kafka made stream processing so successful in modern businesses—it allows capturing and replaying a stream of events. Without this capability, stream processing would not be more than a lab toy for data scientists.


**Stream processing**

This is a contentious and nonblocking option. Filling the gap between the request-response world where we wait for events that take two milliseconds to process and the batch processing world where data is processed once a day and takes eight hours to complete. Most business processes don’t require an immedi‐ ate response within milliseconds but can’t wait for the next day either. Most busi‐ ness processes happen continuously, and as long as the business reports are updated continuously and the line of business apps can continuously respond, the processing can proceed without anyone waiting for a specific response within milliseconds. Business processes like alerting on suspicious credit transactions or network activity, adjusting prices in real-time based on supply and demand, or tracking deliveries of packages are all natural fit for continuous but nonblocking processing.

It is important to note that the definition doesn’t mandate any specific framework, API, or feature. As long as you are continuously reading data from an unbounded dataset, doing something to it, and emitting output, you are doing stream processing. But the processing has to be continuous and ongoing. A process that starts every day at 2:00 A.M., reads 500 records from the stream, outputs a result, and goes away doesn’t quite cut it as far as stream processing goes.

## Stream-Processing Concepts

### Time
**Event time**

This is the time the events we are tracking occurred and the record was created— the time a measurement was taken, an item at was sold at a shop, a user viewed a page on our website, etc. In versions 0.10.0 and later, Kafka automatically adds the current time to producer records at the time they are created. If this does not match your application’s notion of event time, such as in cases where the Kafka record is created based on a database record some time after the event occurred, you should add the event time as a field in the record itself. Event time is usually the time that matters most when processing stream data.

**Log append time**

This is the time the event arrived to the Kafka broker and was stored there. In versions 0.10.0 and higher, Kafka brokers will automatically add this time to records they receive if Kafka is configured to do so or if the records arrive from older producers and contain no timestamps. This notion of time is typically less relevant for stream processing, since we are usually interested in the times the events occurred. For example, if we calculate number of devices produced per day, we want to count devices that were actually produced on that day, even if there were network issues and the event only arrived to Kafka the following day. However, in cases where the real event time was not recorded, log append time can still be used consistently because it does not change after the record was created.

**Processing time**

This is the time at which a stream-processing application received the event in order to perform some calculation. This time can be milliseconds, hours, or days after the event occurred. This notion of time assigns different timestamps to the same event depending on exactly when each stream processing application hap‐ pened to read the event. It can even differ for two threads in the same applica‐ tion! Therefore, this notion of time is highly unreliable and best avoided.

### State
As long as you only need to process each event individually, stream processing is a very simple activity. For example, if all you need to do is read a stream of online shop‐ ping transactions from Kafka, find the transactions over $10,000 and email the rele‐ vant salesperson, you can probably write this in just few lines of code using a Kafka consumer and SMTP library.

Stream processing becomes really interesting when you have operations that involve multiple events: counting the number of events by type, moving averages, joining two streams to create an enriched stream of information, etc. In those cases, it is not enough to look at each event by itself; you need to keep track of more information— how many events of each type did we see this hour, all events that require joining, sums, averages, etc. We call the information that is stored between events a state.

It is often tempting to store the state in variables that are local to the stream- processing app, such as a simple hash-table to store moving counts. In fact, we did just that in many examples in this book. However, this is not a reliable approach for managing state in stream processing because when the stream-processing application is stopped, the state is lost, which changes the results. This is usually not the desired outcome, so care should be taken to persist the most recent state and recover it when starting the application.
Stream processing refers to several types of state:

**Local or internal state**

State that is accessible only by a specific instance of the stream-processing appli‐ cation. This state is usually maintained and managed with an embedded, in- memory database running within the application. The advantage of local state is that it is extremely fast. The disadvantage is that you are limited to the amount of memory available. As a result, many of the design patterns in stream processing focus on ways to partition the data into substreams that can be processed using a limited amount of local state.

**External state**

State that is maintained in an external datastore, often a NoSQL system like Cas‐ sandra. The advantages of an external state are its virtually unlimited size and the fact that it can be accessed from multiple instances of the application or even from different applications. The downside is the extra latency and complexity introduced with an additional system. Most stream-processing apps try to avoid having to deal with an external store, or at least limit the latency overhead by caching information in the local state and communicating with the external store as rarely as possible. This usually introduces challenges with maintaining consis‐ tency between the internal and external state.

### Stream-Table Duality
We are all familiar with database tables. A table is a collection of records, each identi‐ fied by its primary key and containing a set of attributes as defined by a schema. Table records are mutable (i.e., tables allow update and delete operations). Querying a table allows checking the state of the data at a specific point in time. For example, by querying the CUSTOMERS_CONTACTS table in a database, we expect to find cur‐ rent contact details for all our customers. Unless the table was specifically designed to include history, we will not find their past contacts in the table.

Unlike tables, streams contain a history of changes. Streams are a string of events wherein each event caused a change. A table contains a current state of the world, which is the result of many changes. From this description, it is clear that streams and tables are two sides of the same coin—the world always changes, and sometimes we are interested in the events that caused those changes, whereas other times we are interested in the current state of the world. Systems that allow you to transition back and forth between the two ways of looking at data are more powerful than systems that support just one.

In order to convert a table to a stream, we need to capture the changes that modify the table. Take all those insert, update, and delete events and store them in a stream. Most databases offer change data capture (CDC) solutions for capturing these changes and there are many Kafka connectors that can pipe those changes into Kafka where they will be available for stream processing.

In order to convert a stream to a table, we need to apply all the changes that the stream contains. This is also called materializing the stream. We create a table, either in memory, in an internal state store, or in an external database, and start going over all the events in the stream from beginning to end, changing the state as we go. When we finish, we have a table representing a state at a specific time that we can use.

### Time Windows
Most operations on streams are windowed operations—operating on slices of time: moving averages, top products sold this week, 99th percentile load on the system, etc. Join operations on two streams are also windowed—we join events that occurred at the same slice of time. Very few people stop and think about the type of window they want for their operations. For example, when calculating moving averages, we want to know:
* Size of the window: do we want to calculate the average of all events in every five- minute window? Every 15-minute window? Or the entire day? Larger windows are smoother but they lag more—if price increases, it will take longer to notice than with a smaller window.
* How often the window moves (advance interval): five-minute averages can update every minute, second, or every time there is a new event. When the advance interval is equal to the window size, this is sometimes called a tumbling window. When the window moves on every record, this is sometimes called a sliding window.
* How long the window remains updatable: our five-minute moving average calcu‐ lated the average for 00:00-00:05 window. Now an hour later, we are getting a few more results with their event time showing 00:02. Do we update the result for the 00:00-00:05 period? Or do we let bygones be bygones? Ideally, we’ll be able to define a certain time period during which events will get added to their respec‐ tive time-slice. For example, if the events were up to four hours late, we should recalculate the results and update. If events arrive later than that, we can ignore them.

## Stream-Processing Design Patterns

### Single-Event Processing
The most basic pattern of stream processing is the processing of each event in isola‐ tion. This is also known as a map/filter pattern because it is commonly used to filter unnecessary events from the stream or transform each event. (The term “map” is based on the map/reduce pattern in which the map stage transforms events and the reduce stage aggregates them.)

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_stream_p1.png" width="700" height="500">
</div>

### Processing with Local State
Most stream-processing applications are concerned with aggregating information, especially time-window aggregation. An example of this is finding the minimum and maximum stock prices for each day of trading and calculating a moving average.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_stream_p2.png" width="700" height="500">
</div>

### Multiphase Processing/Repartitioning
Local state is great if you need a group by type of aggregate. But what if you need a result that uses all available information? For example, suppose we want to publish the top 10 stocks each day—the 10 stocks that gained the most from opening to clos‐ ing during each day of trading. Obviously, nothing we do locally on each application instance is enough because all the top 10 stocks could be in partitions assigned to other instances. What we need is a two-phase approach. First, we calculate the daily gain/loss for each stock symbol. We can do this on each instance with a local state. Then we write the results to a new topic with a single partition. This partition will be read by a single application instance that can then find the top 10 stocks for the day. The second topic, which contains just the daily summary for each stock symbol, is obviously much smaller with significantly less traffic than the topics that contain the trades themselves, and therefore it can be processed by a single instance of the application.

This type of multiphase processing is very familiar to those who write map-reduce code, where you often have to resort to multiple reduce phases. If you’ve ever written map-reduce code, you’ll remember that you needed a separate app for each reduce step. Unlike MapReduce, most stream-processing frameworks allow including all steps in a single app, with the framework handling the details of which application instance (or worker) will run reach step.


<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_stream_p3.png" width="700" height="500">
</div>

### Processing with External Lookup: Stream-Table Join




# Question:
1. 怎么rebalance的, 在poll的过程中, partition down了, poll不到然后就自动换了?


