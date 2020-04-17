# Design, Building And Deploying Messaging Solutions
主要讲述如何构建一个消息系统

# Basic Concept

## What is Messaging
Messaging is a technology that enables high-speed, asynchronous, program-to-program communication with reliable delivery. Programs communicate by sending packets of data called messages to each other. Channels, also known as queues, are logical pathways that connect the programs and convey messages. A channel behaves like a collection or array of messages, but one that is magically shared across multiple computers and can be used concurrently by multiple applications. A sender or producer is a program that sends a message by writing the message to a channel. A receiver or consumer is a program that receives a message by reading (and deleting) it from a channel.

The message itself is simply some sort of data structure—such as a string, a byte array, a record, or an object. It can be interpreted simply as data, as the description of a command to be invoked on the receiver, or as the description of an event that occurred in the sender. 

## What is Message System
It manage the message to be sent, transfered and recieved like the way that the database manages the data.

**Advantage of message system**

The reason a messaging system is needed to move messages from one computer to another is that computers and the networks that connect them are inherently unreliable. Just because one application is ready to send a communication does not mean that the other application is ready to receive it. Even if both applications are ready, the network may not be working, or may fail to transmit the data properly. A messaging system overcomes these limitations by repeatedly trying to transmit the message until it succeeds. Under ideal circumstances, the message is transmitted successfully on the first try, but circumstances are often not ideal.

In essence, a message is transmitted in five steps:
1. Create — The sender creates the message and populates it with data.
2. Send — The sender adds the message to a channel.
3. Deliver — The messaging system moves the message from the sender’s computer to the
receiver’s computer, making it available to the receiver.
4. Receive — The receiver reads the message from the channel.
5. Process — The receiver extracts the data from the message.

**两个重要特性**

1. **Send and forget**— In step 2, the sending application sends the message to the message channel. Once that send is complete, the sender can go on to other work while the messaging system transmits the message in the background. The sender can be confident that the receiver will eventually receive the message and does not have to wait until that happens.
2. **Store and forward** — In step 2, when the sending application sends the message to the message channel, the messaging system stores the message on the sender’s computer, either in memory or on disk. In step 3, the messaging system delivers the message by forwarding it from the sender’s computer to the receiver’s computer, and then stores the message once again on the receiver’s computer. This store-and-forward process may be repeated many times, as the message is moved from one computer to another, until it reaches the receiver’s computer.


## Why use message?
* **Remote Communication.** Messaging enables separate applications to communicate and transfer data. Two objects that reside in the same process can simply share the same data in memory. Sending data to another computer is a lot more complicated and requires data to be copied from one computer to another. This means that objects have to "serializable", i.e. they can be converted into a simple byte stream that can be sent across the network.
* **Platform/Language Integration.** When connecting multiple computer systems via remote communication, these systems likely use different languages, technologies and platforms, perhaps because they were developed over time by independent teams.  In these circumstances, a messaging system can be a universal translator between the applications that works with each one’s language and platform on its own terms, yet allows them to all communicate through a common messaging paradigm. 
* **Asynchronous Communication.** Messaging enables a send and forget approach to communication. The sender does not have to wait for the receiver to receive and process the message; it does not even have to wait for the messaging system to deliver the message.
* **Variable Timing.** With synchronous communication, the caller must wait for the receiver to finish processing the call before the caller can receive the result and continue. In this way, the caller can only make calls as fast as the receiver can perform them. On the other hand, asynchronous communication allows the sender to batch requests to the receiver at its own pace, and for the receiver to consume the requests at its own different pace. This allows both applications to run at maximum throughput and not waste time waiting on each other.
* **Throttling.** 其实和上面是差不多意思. A problem with remote procedure calls is that too many of them on a single receiver at the same time can overload the receiver. This can cause performance degradation and even cause the receiver to crash. Asynchronous communication enables the receiver to control the rate at which it consumes requests, so as not to become overloaded by too many simultaneous requests. The adverse effect on callers caused by this throttling is minimized because the communication is asynchronous, so the callers are not blocked waiting on the receiver.
* **Reliable Communication.** 这点有待研究, 相比于RPC, 如果是链式调用的话, message传递会遇到网络问题, 在传递中信息丢失会导致整个传递失败. 而消息系统则采用store and forward, 如果消息在某一环丢失, 则可以在原地冲洗传递而不用重新经过整个传递过程, 类似于kafka会存储topic
* **Disconnected Operation.** 能够容许网络失联的情况
* **Mediation.** The messaging system acts as a mediator—as in the Mediator pattern —between all of the programs that can send and receive messages. An application can use it as a directory of other applications or services available to integrate with. If an application becomes disconnected from the others, it need only reconnect to the messaging system, not to all of the other messaging applications. The messaging system can be used to provide a high number of distributed connections to a shared resource, such as a database. The messaging system can employ redundant resources to provide high-availability, balance load, reroute around failed network connections, and tune performance and quality of service.

## Challenges of Asynchronous Messaging
1. **Complex programming model. **
2. **Sequence issues. **
3. **Synchronous scenarios.** Not all applications can operate in a send and forget mode. If a user is looking for airline tickets, he or she is going to want to see the ticket price right away, not after some undetermined time. Therefore, many messaging systems need to bridge the gap between synchronous and asynchronous solutions. 
4.  **Performance.** Messaging systems do add some overhead to communication. It takes effort to make data into a message and send it, and to receive a message and process it. If you have to transport a huge chunk of data, dividing it into a gazillion small pieces may not be a smart idea. For example, if an integration solution needs to synchronize information between two exiting systems, the first step is usually to replicate all relevant information from one system to the other. For such a bulk data replication step, ETL (extract, transform, and load) tools are much more efficient than messaging. Messaging is best suited to keeping the systems in sync after the initial data replication.

# Integration Styles

## Application Integration Criteria
一个优秀的系统需要考虑哪几个方面?
1. **Application coupling** 减少依赖
2. **Integration simplicity** minimize changing
3. **Integration technology** Different integration techniques require varying amounts of specialized software and hardware. These special tools can be expensive, can lead to vendor lock-in, and increase the burden on developers to understand how to use the tools to integrate applications.
4. **Data format**
5. **Data timeliness** Latency in data sharing has to be factored into the integration design; 
   
## Application Integration Options
* **File Transfer** — Have each application produce files of shared data for others to consume, and consume files that others have produced.
* **Shared Database** — Have the applications store the data they wish to share in a common database.
* **Remote Procedure Invocation** — Have each application expose some of its procedures so that they can be invoked remotely, and have applications invoke those to run behavior and exchange data.
* **Messaging** — Have each application connect to a common messaging system, and exchange data and invoke behavior using messages.

## File Transfer
In an ideal world, you might imagine an organization operating from a single cohesive piece of software, designed from the beginning to work in a unified and coherent way. Of course even the smallest operations don't work like that. Multiple pieces of software handle different aspects of the enterprise. This is due to a host of reasons.
People buy packages that are developed by outside organizations
Different system are built at different times, leading to different technology choices
Different systems are built by different people, whose experience and preferences lead them to different approaches to building applications
Getting an application out and delivering value is more important than ensuring that integration is addressed, especially when that integration isn't adding any value to the application under development.

The great advantage of files is that integrators need no knowledge of the internals of an application. The application team itself usually provides the file. The file's contents and format are negotiated with integrators, although if a package is used there's often limited choices. The integrators then deal with the transformations required for other applications, or they leave it up the consuming applications to decide how they want to manipulate and read the file.

One of the most obvious issues with File Transfer is that updates tend to occur infrequently, as a result systems can get out of synchronization. 

## Shared Database
One of the biggest difficulties with Shared Database is coming up with a suitable design for the shared database. Coming up with a unified schema that can meet the needs of multiple applications is a very difficult exercise, often resulting in a schema that application programmers find difficult to work with.

Another, harder limit to Shared Database is external packages. Often they won't work with a schema other than their own. 

Multiple applications using a Shared Database to frequently read and modify the same data can cause performance bottlenecks and even deadlocks as each application locks others out of the data. When the applications are distributed across multiple computers, the database must be distributed as well so that each application can access the database locally, which confuses the issue of which computer the data should be stored on. A distributed database with locking conflicts can easily become a performance nightmare.

## Remote Procedure Invocation
focus on functionalities

The advantage of RPC is its encapsulated data

Although the encapsulation helps reduce the coupling of the applications, by eliminating a large shared data structure, the applications are still fairly tightly coupled together. The remote calls each system supports tends to tie the different systems into a growing knot. In particular, sequencing--doing certain things in a particular order--can make it difficult to change systems independently.

## Messaging
File Transfer and Shared Database enable applications to share their data, but not their functionality. Remote Procedure Invocation enables applications to share functionality, but tightly couples them in the process.

File Transfer allows you keep the applications very well decoupled, but at the cost of timeliness. Systems just can't keep up with each other. Collaborative behavior is way too slow. Shared Database keeps data together in a responsive way, but at the cost of coupling everything to the database. It also fails to handle collaborative behavior.

Faced with these problems, Remote Procedure Invocation seems an appealing choice. But extending a model used for a single application to application integration runs into plenty of other weaknesses. These weaknesses start with the essential problems of distributed development. Despite the fact that remote procedure calls look like local calls, they don't act the same. Remote calls are slower, and can fail. With multiple applications communicating across an enterprise, you don't want one application's failure to bring down all of the other applications. Also, you don't want to design a system assuming that calls are fast and you don't want each application knowing details of other applications, even if it's only details about their interfaces.

What we need is something like File Transfer where lots of little data packets can be produced quickly, transferred easily, and the receiver application is automatically notified when a new packet is available for consumption. The transfer needs a retry mechanism to make sure it succeeds. The details of any disk structure or database for storing the data needs to be hidden from the applications so that, unlike Shared Database, the storage schema and details can be easily changed to reflect the changing needs of the enterprise. One application should be able to send a packet of data to another application to invoke behavior in the other application, like Remote Procedure Invocation, but without being prone to failure. The data transfer should be asynchronous so that the sender does not need to wait on the receiver, especially when retry is necessary.

# Message System

## Basic Messaging Concepts

**Channels** — Messaging applications transmit data through a Message Channel, a virtual pipe that connects a sender to a receiver. A newly installed messaging system doesn’t contain any channels; you must determine how your applications need to communicate and then create the channels to facilitate it.

**Messages** — A Message is an atomic packet of data that can be transmitted on a channel. Thus to transmit data, an application must break the data into one or more packets, wrap each packet as a message, and then send the message on a channel. Likewise, a receiver application receives a message and must extract the data from the message to process it. The message system will try repeatedly to deliver the message (e.g., transmit it from the sender to the receiver) until it succeeds.

**Multi-step delivery** — In the simplest case, the message system delivers a message directly from the sender’s computer to the receiver’s computer. However, actions often need to be performed on the message after it is sent by its original sender but before it is received by its final receiver. For example, the message may have to be validated or transformed because the receiver expects a different message format than the sender. A Pipes and Filters architecture describes how multiple processing steps can be chained together using channels.

**Routing** — In a large enterprise with numerous applications and channels to connect them, a message may have to go through several channels to reach its final destination. The route a message must follow may be so complex that the original sender does not know what channel will get the message to the final receiver. Instead, the original sender sends the message to a Message Router, an application component and filter in the pipes-and-filters architecture, which will determine how to navigate the channel topology and direct the message to the final receiver, or at least to the next router.

**Transformation** — Various applications may not agree on the format for the same conceptual data; the sender formats the message one way, yet the receiver expects it to be formatted another way. To reconcile this, the message must go through an intermediate filter, a Message Translator, that converts the message from one format to another.

**Endpoints** — An application does not have some built-in capability to interface with a messaging system. Rather, it must contain a layer of code that knows both how the application works and how the messaging system works, bridging the two so that they work together. This bridge code is a set of coordinated Message Endpoints that enable the application to send and receive messages.

# Message Channel

1. Point to Point Channel 信息只会被一个reciever接收
2. Publish-Subscribe Channel 有类似broadcast功能的
3. Datatype Channel 就是什么类型走什么channel, 一个channel只能有一个类型
4. Invalid Message Channel The receiver should move the improper message to an Invalid Message Channel, a special channel for messages that could not be processed by their receivers. 但这个channel不是处理application error的, 好比删除一个不存在的数据的command message, 这是application error