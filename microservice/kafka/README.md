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
[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/kafka_producer.png)
We start producing messages to Kafka by creating a ProducerRecord, which must include the topic we want to send the record to and a value. Optionally, we can also specify a key and/or a partition. Once we send the ProducerRecord, the first thing the producer will do is serialize the key and value objects to ByteArrays so they can be sent over the network.

Next, the data is sent to a partitioner. If we specified a partition in the ProducerRecord, the partitioner doesn’t do anything and simply returns the partition we specified. If we didn’t, the partitioner will choose a partition for us, usually based on the ProducerRecord key. Once a partition is selected, the producer knows which topic and partition the record will go to. It then adds the record to a batch of records that will also be sent to the same topic and partition. A separate thread is responsible for sending those batches of records to the appropriate Kafka brokers.

When the broker receives the messages, it sends back a response. If the messages were successfully written to Kafka, it will return a RecordMetadata object with the topic, partition, and the offset of the record within the partition. If the broker failed to write the messages, it will return an error. When the producer receives an error, it may retry sending the message a few more times before giving up and returning an error.

### Constructing a Kafka Producer
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
* Fire-and-forget We send a message to the server and don’t really care if it arrives succesfully or not. Most of the time, it will arrive successfully, since Kafka is highly available and the producer will retry sending messages automatically. However, some mes‐ sages will get lost using this method.
* Synchronous send We send a message, the send() method returns a Future object, and we use get() to wait on the future and see if the send() was successful or not.
* Asynchronous send We call the send() method with a callback function, which gets triggered when it receives a response from the Kafka broker.

### 一些常用参数
**acks**

The acks parameter controls how many partition replicas must receive the record before the producer can consider the write successful. If **acks=0**, the producer will not wait for a reply from the broker before assuming the message was sent successfully. If **acks=1**, the producer will receive a success response from the broker the moment the leader replica received the message. If **acks=all**, the producer will receive a success response from the broker once all in-sync replicas received the message.

**buffer.memory**

This sets the amount of memory the producer will use to buffer messages waiting to be sent to brokers. If messages are sent by the application faster than they can be delivered to the server, the producer may run out of space.

**retries**

**batch.size**

**linger.ms**

**client.id**

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


### Sending a Message to Kafka
```java
ProducerRecord<String, String> record = new ProducerRecord<>("CustomerCountry", "Precision Products", "France");
try {
    producer.send(record);
    //producer.send(record).get();
} catch (Exception e) {
    e.printStackTrace();
}


//send asynchronously
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

# Creating a Kafka Consumer
```java
Properties props = new Properties();
props.put("bootstrap.servers", "broker1:9092,broker2:9092");
props.put("group.id", "CountryCounter");
props.put("key.deserializer",
    "org.apache.kafka.common.serialization.StringDeserializer");
props.put("value.deserializer",
    "org.apache.kafka.common.serialization.StringDeserializer");
KafkaConsumer<String, String> consumer = new KafkaConsumer<String,
String>(props);

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

You can’t have multiple consumers that belong to the same group in one thread and you can’t have multiple threads safely use the same consumer. One consumer per thread is the rule. To run mul‐ tiple consumers in the same group in one application, you will need to run each in its own thread. It is useful to wrap the con‐ sumer logic in its own object and then use Java’s ExecutorService to start multiple threads each with its own consumer. The Conflu‐ ent blog has a [tutorial](https://www.confluent.io/blog/tutorial-getting-started-with-the-new-apache-kafka-0-9-consumer-client/) that shows how to do just that.

### Commits and Offsets

Whenever we call poll(), it returns records written to Kafka that consumers in our group have not read yet. This means that we have a way of tracking which records were read by a consumer of the group. As discussed before, one of Kafka’s unique characteristics is that it does not track acknowledgments from consumers the way many JMS queues do. Instead, it allows consumers to use Kafka to track their posi‐ tion (offset) in each partition.
We call the action of updating the current position in the partition a commit.

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
        consumer.commitAsync();
    }
} catch (Exception e) {
    log.error("Unexpected error", e);
} finally {
    try {
        consumer.commitSync();
    } finally {
        consumer.close();
    }
}
```

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

