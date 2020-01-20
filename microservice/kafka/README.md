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
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/1.png" width="700" height="500">
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