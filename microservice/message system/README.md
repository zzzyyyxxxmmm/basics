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
