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

### Close Consumer
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

