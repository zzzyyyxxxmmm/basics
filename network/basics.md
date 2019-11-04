# OSI
[seven layers](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/OSI.png)
值得一提的是, socket是作用在application layer和transportation layer中间的一个抽象层
socket is the interface between the application process and the transport-layer protocol.

# TCP vs UDP

### Reliable Data Transfer
Packets can get lost within a computer network. For example, a packet can overflow a buffer in a router, or can be discarded by a host or router after having some of its bits corrupted. For many applications—such as electronic mail, file transfer, remote host access, Web document transfers, and financial applications—data loss can have devastating consequences. Thus, to support these applications, something has to be done to guarantee that the data sent by one end of the appli- cation is delivered correctly and completely to the other end of the application. If a protocol provides such a guaranteed data delivery service, it is said to provide reliable data transfer. One important service that a transport-layer protocol can potentially provide to an application is process-to-process reliable data transfer. When a transport protocol provides this service, the sending process can just pass its data into the socket and know with complete confidence that the data will arrive without errors at the receiving process.

When a transport-layer protocol doesn’t provide reliable data transfer, some of the data sent by the sending process may never arrive at the receiving process. This may be acceptable for loss-tolerant applications, most notably multimedia applica- tions such as conversational audio/video that can tolerate some amount of data loss. In these multimedia applications, lost data might result in a small glitch in the audio/video—not a crucial impairment.

### Throughput
In Chapter 1 we introduced the concept of available throughput, which, in the con- text of a communication session between two processes along a network path, is the rate at which the sending process can deliver bits to the receiving process. Because other sessions will be sharing the bandwidth along the network path, and because these other sessions will be coming and going, the available throughput can fluctuate with time. These observations lead to another natural service that a transport-layer protocol could provide, namely, guaranteed available throughput at some specified rate. With such a service, the application could request a guaranteed throughput of r bits/sec, and the transport protocol would then ensure that the available throughput is always at least r bits/sec. Such a guaranteed through- put service would appeal to many applications. For example, if an Internet teleph- ony application encodes voice at 32 kbps, it needs to send data into the network and have data delivered to the receiving application at this rate. If the transport protocol cannot provide this throughput, the application would need to encode at a lower rate (and receive enough throughput to sustain this lower coding rate) or may have to give up, since receiving, say, half of the needed throughput is of little or no use to this Internet telephony application. Applications that have throughput requirements are said to be **bandwidth-sensitive applications**. Many current multimedia applications are bandwidth sensitive, although some multimedia applications may use adaptive coding techniques to encode digitized voice or video at a rate that matches the currently available throughput.

While bandwidth-sensitive applications have specific throughput requirements, **elastic applications** can make use of as much, or as little, throughput as happens to be available. Electronic mail, file transfer, and Web transfers are all elastic applications. Of course, the more throughput, the better. There’s an adage that says that one cannot be too rich, too thin, or have too much throughput!

### Security
Finally, a transport protocol can provide an application with one or more security services. For example, in the sending host, a transport protocol can encrypt all data transmitted by the sending process, and in the receiving host, the transport-layer protocol can decrypt the data before delivering the data to the receiving process.

### Timing
A transport-layer protocol can also provide timing guarantees. Such a service would be appealing to interactive real-time applications, such as Internet telephony.

| Application                            | Data Loss     | Throughput                                  | Time-Sensitive    |
|----------------------------------------|---------------|---------------------------------------------|-------------------|
| File transfer/download                 | No loss       | Elastic                                     | No                |
| E-mail                                 | No loss       | Elastic                                     | No                |
| Web documents                          | No loss       | Elastic (few kbps)                          | No                |
| Internet telephony/ Video conferencing | Loss-tolerant | Audio: few kbps–1Mbps Video: 10 kbps–5 Mbps | Yes: 100s of msec |
| Streaming stored audio/video           | Loss-tolerant | Same as above                               | Yes: few seconds  |
| Interactive games                      | Loss-tolerant | Few kbps–10 kbps                            | Yes: 100s of msec |
| Instant messaging                      | No loss       | Elastic                                     | Yes and no        |


## TCP
The TCP service model includes a **connection-oriented service** and a **reliable data transfer service**.

**Connection-oriented service**

TCP has the client and server exchange transport- layer control information with each other before the application-level messages begin to flow. This so-called handshaking procedure alerts the client and server, allowing them to prepare for an onslaught of packets. After the handshaking phase, a TCP connection is said to exist between the sockets of the two processes. The connection is a full-duplex connection in that the two processes can send messages to each other over the connection at the same time. When the application finishes sending messages, it must tear down the connection.

**Reliable data transfer service**

The communicating processes can rely on TCP to deliver all data sent without error and in the proper order. When one side of the application passes a stream of bytes into a socket, it can count on TCP to deliver the same stream of bytes to the receiving socket, with no missing or duplicate bytes.

**congestion-control mechanism**

### Building a Reliable Data Transfer Protocol
1. 假设我们有一个sender和一个reciever, 保证网络可靠, 即packet不会被损坏, 也不会丢失, 那么直接发送即可
2. 假设packet会损坏, 那么我们需要检测error和recover from error. 检测error需要在packet里加上checksum, recover from error就是重发, reciever通过checksum检测发送过来的包是否损坏, 如果损坏, 则发送NAK, 否则发送ACK. sender根据接收到的决定是否重发
3. 再假设, 发送过来的ACK/NAK也损坏了呢? 最直接的想法就是对这个ACK/NAK也建立一个关于ACK/NAK的反馈, 但这样显然会无限循环了.(两座山的故事). 正确方法就是也检测ACK/NAK是否损坏, 损坏或者NAK就直接重发. 那reciever就有可能接收到duplicate packet, 区分方法就是加上sequential number: 0代表第一次发送, 1代表重发.
4. 如果packet发送的时候丢了呢. 这里加上timer计时即可

以上都是基于 stop-and-wait的, 一次只能发送一个, 我们需要基于pipline式的, 一次支持发送多条, 因此上面的方法需要进一步改动:
* The range of sequence numbers must be increased, since each in-transit packet (not counting retransmissions) must have a unique sequence number and there may be multiple, in-transit, unacknowledged packets.
* The sender and receiver sides of the protocols may have to buffer more than one packet. Minimally, the sender will have to buffer packets that have been trans- mitted but not yet acknowledged. Buffering of correctly received packets may also be needed at the receiver, as discussed below.
* The range of sequence numbers needed and the buffering requirements will depend on the manner in which a data transfer protocol responds to lost, corrupted, and overly delayed packets. Two basic approaches toward pipelined error recovery can be identified: **Go-Back-N** and **selective repeat**.

Sequential Number 是根据窗口大小循环的鸭:stuck_out_tongue_winking_eye:~

### Go-Back-N
[view](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/GBN.png)

发送方maintain一个大小为N的窗口,类似于一个buffer发送packet. 

接收方窗口大小为1, reciever必须要按续接收, 比如上一个接收的是n, 下一个一定要接收n+1, 否则则会要求sender重发n之后的所有packet

缺点: 已发送的正确packet可能会重发

### Selective Repeat (SR)
[view](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/SR.png)

接收窗口的尺寸不能超过序号范围的1/2，否则可能造成帧的重叠。另外，发送窗口的尺寸一般和接收窗口的尺寸相同，发送端为每一个发送缓存设置一个定时计数器，定时器一旦超时，相应输出缓存区中的帧就被重发。

This is to avoid packets being recognized incorrectly.

If the windows size is greater than half the sequence number space, then if an ACK is lost, the sender may send new packets that the receiver believes are retransmissions.

**Why is window size less than or equal to half the sequence number in SR protocol?**
```
For example, if our sequence number range is 0-3 and the window size is 3, this situation can occur.

A -> 0 -> B
A -> 1 -> B
A -> 2 -> B
A <- 2ack <- B (this is lost)
A -> 0 -> B
A -> 1 -> B
A -> 2 -> B

After the lost packet, B now expects the next packets to have sequence numbers 3, 0, and 1.
But, the 0 and 1 that A is sending are actually retransmissions, so B receives them out of order.
By limiting the window size to 2 in this example, we avoid this problem because B will be expecting 2 and 3, and only 0 and 1 can be retransmissions.
```

## TCP Segment Structure
[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/tcp_segment.png)

Both Ethernet and PPP link-layer protocols have an maximum segment size of 1,500 bytes. 

Because the TCP header is typically 20 bytes (12 bytes more than the UDP header), segments sent by Telnet may be only 21 bytes in length.

Figure 3.29 shows the structure of the TCP segment. As with UDP, the header includes source and destination port numbers, which are used for multiplexing/demultiplexing data from/to upper-layer applications. Also, as with UDP, the header includes a checksum field. A TCP segment header also contains the following fields:
* The 32-bit sequence number field and the 32-bit acknowledgment number field are used by the TCP sender and receiver in implementing a reliable data transfer service, as discussed below.
* The 16-bit receive window field is used for flow control. We will see shortly that it is used to indicate the number of bytes that a receiver is willing to accept.
* The 4-bit header length field specifies the length of the TCP header in 32-bit words. The TCP header can be of variable length due to the TCP options field. (Typically, the options field is empty, so that the length of the typical TCP header is 20 bytes.)
* The optional and variable-length options field is used when a sender and receiver negotiate the maximum segment size (MSS) or as a window scaling fac- tor for use in high-speed networks. A time-stamping option is also defined. See RFC 854 and RFC 1323 for additional details.
* The flag field contains 6 bits. The ACK bit is used to indicate that the value car- ried in the acknowledgment field is valid; that is, the segment contains an acknowledgment for a segment that has been successfully received. The RST, SYN, and FIN bits are used for connection setup and teardown, as we will dis- cuss at the end of this section. Setting the PSH bit indicates that the receiver should pass the data to the upper layer immediately. Finally, the URG bit is used to indicate that there is data in this segment that the sending-side upper-layer entity has marked as “urgent.” The location of the last byte of this urgent data is indicated by the 16-bit urgent data pointer field. TCP must inform the receiving-side upper-layer entity when urgent data exists and pass it a pointer to the end of the urgent data. (In practice, the PSH, URG, and the urgent data pointer are not used. However, we mention these fields for completeness.)

## telnet example
[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/telnet.png)

## timeout的时间设置为多少比较合理
**EstimatedRTT = (1 – α) • EstimatedRTT + α • SampleRTT**

α = 0.125

The sample RTT, denoted **SampleRTT**, for a segment is the amount of time between when the segment is sent (that is, passed to IP) and when an acknowledg- ment for the segment is received. Obviously, the SampleRTT values will fluctuate from segment to segment due to congestion in the routers and to the varying load on the end systems. Because of this fluctuation, any given SampleRTT value may be atypical. In order to estimate a typical RTT, it is therefore natural to take some sort of average of the Sam- pleRTT values. TCP maintains an average, called **EstimatedRTT**, of the SampleRTT values.


**DevRTT = (1-β) * DevRTT + β * |SampleRTT - EstimatedRTT|**

第一次的DevRTT=1/2(SampleRTT)，以后按公式来计算，推荐β为0.25

**TimeoutInterval = EstimatedRTT + 4*DevRTT** 

Also, when a timeout occurs, the value of TimeoutInterval is doubled to avoid a premature timeout occurring for a subsequent segment that will soon be acknowledged. However, as soon as a segment is received and EstimatedRTT is updated, the TimeoutInterval is again computed using the formula above.

推荐的初始TimeoutInterval为1秒。出现超时后，TimeoutInterval直接加倍。因为此次重传可能是报文确认ACK因为网络拥塞而延迟到达从而导致报文重传，重传报文后，不久，ACK到达，会导致SampleRTT变小，进而使TimeoutInterval变小，使后面的报文出现过早超时！

### Fast Retransmit
[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/fast_retransmit.png)
首先对于接收方来说，如果接收方收到一个失序的报文段，就立即回送一个 ACK 给发送方，而不是等待发送延时的 ACK. 这样做的目的是可以让发送方尽可能早的知道报文段. 快重传算法规定，如果发送方一连收到 3 个重复的确认，就应当立即传送对方未收到的报文 M3，而不必等待 M3 的重传计时器到期。


————————————————
版权声明：本文为CSDN博主「--Allen--」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/q1007729991/article/details/70185266

### three-way handshake
[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/three-way.png)

[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/four-way.png)
* Step 1. The client-side TCP first sends a special TCP segment to the server-side TCP. This special segment contains no application-layer data. But one of the flag bits in the segment’s header (see Figure 3.29), the SYN bit, is set to 1. For this reason, this special segment is referred to as a SYN segment. In addition, the client randomly chooses an initial sequence number (client_isn) and puts this number in the sequence number field of the initial TCP SYN segment. This segment is encapsulated within an IP datagram and sent to the server. There has 
been considerable interest in properly randomizing the choice of the client_isn in order to avoid certain security attacks.
* Step 2. Once the IP datagram containing the TCP SYN segment arrives at the server host (assuming it does arrive!), the server extracts the TCP SYN segment from the datagram, allocates the TCP buffers and variables to the connection, and sends a connection-granted segment to the client TCP. (We’ll see in Chapter 8 that the allocation of these buffers and variables before completing the third step of the three-way handshake makes TCP vulnerable to a denial-of-service attack known as SYN flooding.) This connection-granted segment also contains no application- layer data. However, it does contain three important pieces of information in the segment header. First, the SYN bit is set to 1. Second, the acknowledgment field of the TCP segment header is set to client_isn+1. Finally, the server chooses its own initial sequence number (server_isn) and puts this value in the sequence number field of the TCP segment header. This connection-granted segment is saying, in effect, “I received your SYN packet to start a connection with your initial sequence number, client_isn. I agree to establish this con- nection. My own initial sequence number is server_isn.” The connection- granted segment is referred to as a SYNACK segment.
* Step 3. Upon receiving the SYNACK segment, the client also allocates buffers and variables to the connection. The client host then sends the server yet another segment; this last segment acknowledges the server’s connection-granted seg- ment (the client does so by putting the value server_isn+1 in the acknowl- edgment field of the TCP segment header). The SYN bit is set to zero, since the connection is established. This third stage of the three-way handshake may carry client-to-server data in the segment payload.

## TCP Congestion Control
this approach raises three questions.

### How does a TCP sender limit the rate at which it sends traffic into its connection

congestion window

LastByteSent – LastByteAcked <= min{cwnd, rwnd}

### How does a TCP sender perceive that there is congestion on the path between itself and the destination

Let us define a “loss event” at a TCP sender as the occurrence of either a timeout or the receipt of three duplicate ACKs from the receiver. 

Having considered how congestion is detected, let’s next consider the more optimistic case when the network is congestion-free, that is, when a loss event doesn’t occur. In this case, acknowledgments for previously unacknowledged segments will be received at the TCP sender. As we’ll see, TCP will take the arrival of these acknowledgments as an indication that all is well—that segments being transmitted into the network are being successfully delivered to the destination—and will use acknowledgments to increase its congestion window size (and hence its transmission rate). Note that if acknowledgments arrive at a relatively slow rate (e.g., if the end-end path has high delay or contains a low-bandwidth link), then the congestion window will be increased at a relatively slow rate. On the other hand, if acknowledgments arrive at a high rate, then the congestion window will be increased more quickly. Because TCP uses acknowledgments to trigger (or clock) its increase in congestion window size, TCP is said to be self-clocking.

### what algorithm should the sender use to change its send rate as a function of perceived end-to-end congestion?
The algorithm has three major components: 
(1) slow start, 
(2) congestion avoidance
(3) fast recovery.

[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/congestion.png)

**Slow Start**

in the slow-start state, the value of cwnd begins at 1 MSS and increases by 1 MSS every time a transmitted segment is first acknowledged. Thus, the TCP send rate starts slow but grows **exponentially** during the slow start phase. 

But when should this exponential growth end? Slow start provides several answers to this question. First, if there is a loss event (i.e., congestion) indicated by a timeout, the TCP sender sets the value of cwnd to 1 and begins the slow start process anew. It also sets the value of a second state variable, ssthresh (shorthand for “slow start threshold”) to cwnd/2—half of the value of the con- gestion window value when congestion was detected. 

**Congestion Avoidance**
达到ssthresh后+1

**Fast Recovery**
3个ACK之后不从慢开始继续, 而是从ssthresh开始+1, 如果只是timeout那还是用慢开始

## UDP
UDP is a no-frills, lightweight transport protocol, providing minimal services. UDP is connectionless, so there is no handshaking before the two processes start to communicate. UDP provides an unreliable data transfer service—that is, when a process sends a message into a UDP socket, UDP provides no guarantee that the message will ever reach the receiving process. Furthermore, messages that do arrive at the receiving process may arrive out of order.


| Application            | Application-Layer Protocol                                   | Underlying Transport Protocol |
|------------------------|--------------------------------------------------------------|-------------------------------|
| Electronic mail        | SMTP [RFC 5321]                                              | TCP                           |
| Remote terminal access | Telnet [RFC 854]                                             | TCP                           |
| Remote terminal access | HTTP [RFC 2616]                                              | TCP                           |
| File transfer          | FTP [RFC 959]                                                | TCP                           |
| Streaming multimedia   | HTTP (e.g., YouTube)                                         | TCP                           |
| Internet telephony     | SIP [RFC 3261], RTP [RFC 3550], or proprietary (e.g., Skype) | UDP or TCP                    |

Because Internet telephony applications (such as Skype) can often tolerate some loss but require a minimal rate to be effective, developers of Inter- net telephony applications usually prefer to run their applications over UDP, thereby circumventing TCP’s congestion control mechanism and packet over- heads. But because many firewalls are configured to block (most types of) UDP traffic, Internet telephony applications often are designed to use TCP as a backup if UDP communication fails.
### UDP segment structure
Source port + Dest port + Length + Checksum + Application data
checksum就是所有数据加到一起 然后反转 010 -> 101, 接收端接收到后求sum, 然后加在一起, 得到111, 全是1, 用来检测信息是否遭到修改
### 选择UDP的原因:
1. Finer application-level control over what data is sent, and when. TCP有拥塞控制, 不能保证发出去的message被准时发出
2. No connection establishment.
3. No connection state. 通常可以比TCP支持更多的client
4. Small packet header overhead. The TCP segment has 20 bytes of header over- head in every segment, whereas UDP has only 8 bytes of overhead.



# HTTP
The HyperText Transfer Protocol (HTTP), the Web’s application-layer protocol, is at the heart of the Web. HTTP is implemented in two programs: a client program and a server program. The client program and server program, executing on different end systems, talk to each other by exchanging HTTP messages. HTTP defines the structure of these messages and how the client and server exchange the messages.

It is important to note that the server sends requested files to clients without stor- ing any state information about the client. If a particular client asks for the same object twice in a period of a few seconds, the server does not respond by saying that it just served the object to the client; instead, the server resends the object, as it has com- pletely forgotten what it did earlier. Because an HTTP server maintains no informa- tion about the clients, HTTP is said to be a **stateless protocol**.

### request 10 images
normally build 5-10 parallel TCP connections

## HTTP Request Message
```
GET /somedir/page.html HTTP/1.1             --request line (method, URL, HTTP version)
Host: www.someschool.edu                    --request header
Connection: close 
User-agent: Mozilla/5.0 
Accept-language: fr
                                            -- blank line
                                            --request body
```

The HEAD method is similar to the GET method. When a server receives a request with the HEAD method, it responds with an HTTP message but it leaves out the requested object. Application developers often use the HEAD method for debug- ging. The PUT method is often used in conjunction with Web publishing tools. It allows a user to upload an object to a specific path (directory) on a specific Web server. The PUT method is also used by applications that need to upload objects to Web servers. The DELETE method allows a user, or an application, to delete an object on a Web server.

## HTTP Response Message
```
HTTP/1.1 200 OK                             --(protocal verion, status code, corresponding status message)            
Connection: close
Date: Tue, 09 Aug 2011 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 09 Aug 2011 15:11:03 GMT Content-Length: 6821
Content-Type: text/html

(data data data data data ...)
```
### http response code
* 1** : 服务器收到请求，需要请求者继续执行操作**

* 2** : 成功**

* 3** : 重定向**

**301 Moved Permanently:** Requested object has been permanently moved; the new URL is specified in Location:header of the response message.The client software will automatically retrieve the new URL.

**304 Not Modified**

* 4** : 客户端错误，请求包含语法错误或无法完成请求

**400 Bad Request:** This is a generic error code indicating that the request could not be understood by the server.

**404 Not Found:** The requested document does not exist on this server.


* 5** : 服务器错误

**505 HTTP Version Not Supported:** The requested HTTP protocol
version is not supported by the server.

## Cache

client first sends a request, it will first check whether the local storage (local storage is established by local ISP)has cache or not. If yes, it just return the cache in response. If not, it will send the request to the server, and when response return, it will be cached for the next time request. The response has a header called Last-Modified which indicate the last time the object was modified.

What if the content they cache is modified on server? The send a request with a header called If-modified-since. The request will go to the server and check if the Last-Modified is same with If-modified-since. If yes, it will return 304 to tell the client to get the cache directly. If not, it will return a normall response and the response will be cached again.

[https建立过程](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/https.png)

# File Transfer: FTP
The most striking **difference** is that FTP uses two parallel TCP connections to transfer a file, a **control connection** and a d**ata connection**. The control connection is used for sending control information between the two hosts—information such as user identification, password, commands to change remote directory, and commands to “put” and “get” files. The data connection is used to actually send a file. 

Thus, with FTP, the control connection remains open throughout the duration of the user session, but a new data connec- tion is created for each file transferred within a session (that is, the data connec- tions are non-persistent).

# P2P

## Torrent内容
1、torrent文件的原理：如果您这个问题是指torrent文件本身，那么，当您对一个文件（或者文件夹）制作成.torrent文件，实际上生成的.torrent文件里面主要包括了这些信息：     
1. 这个文件（文件夹）中数据的SHA1值，比如一个1G的文件，如果按1M每块进行分块，则会被分为了1000块，torrent中就会有这1000个数据块的指纹值（SHA1的hash值），这个占据了torrent文件的绝大部分空间。这些值的目的是为了下载的过程中进行数据校验，确保数据收到的和当时源头制作torrent时的源文件100%一致，防止恶意数据攻击
2. 一般制作torrent文件时，还会要指定一个或者多个Tracker的地址，比如http://www.a.com:8080/announce这种地址。torrent里面一般也会存储了这个信息，这个其实也尤为重要。相当于记录了一个问询服务器的地址，这个问询服务器的作用，后面我再解释。
3. 文件或者文件夹内每个文件的名字，方便下载文件时，磁盘上直接命名好跟原始数据一样的目录结构、文件名。
4. 其它一些辅助和可扩展的信息，比如可以配置一个P2SP的http地址辅助下载，比如制作软件的名字、备注……。
5. 上面信息生成后，torrent会把A）里面的这些信息，以及torrent里面的文件名等关键信息，再进行一次Hash，生成一个新的SHA1值，作为torrent的HASH值，也就是我们经常看到的下载软件里面对这个种子命名的一个唯一的hash值，也有的在magnet这种磁力链接中可以看到这个值，这就是torrent的唯一标记。以上就是.torrent文件的内容，可以用记事本打开，但可能看到乱码。这个文件的编码遵循了bencode编码规则。但实际内容就主要是上面这些。所以，torrent可以理解为对原始数据的一些记录。

## Torrent流程
1. 下载软件拿到.torrent文件后，先进行打开，读取里面的这些信息，载入内存.
2. torrent中有Tracker的地址，下载软件拿到后，会去跟Tracker进行通讯，告诉Tracker：我要下载这个文件（通过hash值作为标记）； Tracker收到请求后，会记录这个客户端的公网IP（记录这厮在下载这个文件），同时呢，会返回给他：我这边还知道哪些人也在下载这个文件，一般是会返回200个IP（如果不够，当然就有多少返回多少）。
当然了，如果下载过程中，协议要求你必须5分钟跟tracker通讯一次，如果太久不通讯，tracker就认为你下线了，会把你从节点列表中删除的。
3. 客户端拿到了一堆IP后，就开始挨个去尝试连接，连上后就开始互相通讯了。比如告诉对方，我有哪些分块，问问对方有哪些，然后把我有的给对方；让对方把他有的某一块给我，这样就你来我往开始了下载。当然，如果很悲催的情况下，此时没别人在线，那就只能没速度了，就只能不停的找啊找啊找朋友，直到找到一个好朋友.
4. 当然，如果torrent中有一个P2SP的Http地址辅助下载，那么也可以同时从这个Http服务器要数据，也会把这个服务器当成一个普通的节点，每次要1块数据，通过Http协议里面的Range标记，指定只要一部分数据过来辅助下载。整个BT的基本原理和过程就是这样，当然，这只是BT的基本原理，要做好一个完善的BT还是有很多路要走的。比如：1）如果Tracker服务器出问题了，连不上这个问询的服务器，就拿不到周围的邻居节点，怎么办？---NB的BT发明者提出了DHT的概念，就算Tracker连不上了，也可以通过分布式哈希表DHT技术，通过DHT网络慢慢的寻找志同道合的邻居节点，只是没有Tracker那么直接那么快速，但慢一些总还是有机会找到邻居的。

## Basic
Each torrent has an infrastructure node called a **tracker**. When a peer joins a torrent, it registers itself with the tracker and periodically informs the tracker that it is still in the torrent. In this manner, the tracker keeps track of the peers that are participating in the torrent. A given torrent may have fewer than ten or more than a thousand peers participating at any instant of time.

when a new peer, Alice, joins the torrent, the tracker randomly selects a subset of peers (for concreteness, say 50) from the set of participat- ing peers, and sends the IP addresses of these 50 peers to Alice. Possessing this list of peers, Alice attempts to establish concurrent TCP connections with all the peers on this list. Let’s call all the peers with which Alice succeeds in establishing a TCP connec- tion “neighboring peers.” As time evolves, some of these peers may leave and other peers (outside the initial 50) may attempt to establish TCP connections with Alice. So a peer’s neighboring peers will fluctuate over time.

At any given time, each peer will have a subset of chunks from the file, with dif- ferent peers having different subsets. Periodically, Alice will ask each of her neighbor- ing peers (over the TCP connections) for the list of the chunks they have. If Alice has L different neighbors, she will obtain L lists of chunks. With this knowledge, Alice will issue requests (again over the TCP connections) for chunks she currently does not have.

So at any given instant of time, Alice will have a subset of chunks and will know which chunks her neighbors have. With this information, Alice will have two important decisions to make. **First, which chunks should she request first from her neighbors? And second, to which of her neighbors should she send requested chunks?** In deciding which chunks to request, Alice uses a technique called rarest first. The idea is to determine, from among the chunks she does not have, the chunks that are the rarest among her neighbors (that is, the chunks that have the fewest repeated copies among her neighbors) and then request those rarest chunks first. In this manner, the rarest chunks get more quickly redistributed, aiming to (roughly) equalize the numbers of copies of each chunk in the torrent.选给自己发送最快的人发送, 并且周期性的更新. 另外会随机选出一个发送,这样新的节点也可以享受加速.

### DHT
[Alice,109.2.321.32]这样的信息被分布式存储在各个peer上
**pair被map到哪个peer上?**
Suppose n = 4 so that all the peer and key identifiers are in the range [0, 15]. Further suppose that there are eight peers in the system with identi- fiers 1, 3, 4, 5, 8, 10, 12, and 15. Finally, suppose we want to store the (key, value) pair (11, Johnny Wu) in one of the eight peers. But in which peer? Using our closest con- vention, since peer 12 is the closest successor for key 11, we therefore store the pair (11, Johnny Wu) in the peer 12. 求closet需要知道其他peer的信息, 所以每个peer都需要维护其他peer的信息, 不合适

### Circular DHT
[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/circularDHT.png)

Using the circular overlay in Figure 2.27(a), now suppose that peer 3 wants to determine which peer in the DHT is responsible for key 11. Using the circular overlay, the origin peer (peer 3) creates a message saying “Who is responsible for key 11?” and sends this message clockwise around the circle. Whenever a peer receives such a mes- sage, because it knows the identifier of its successor and predecessor, it can determine whether it is responsible for (that is, closest to) the key in question. If a peer is not responsible for the key, it simply sends the message to its successor. So, for example, when peer 4 receives the message asking about key 11, it determines that it is not responsible for the key (because its successor is closer to the key), so it just passes the message along to peer 5. This process continues until the message arrives at peer 12, who determines that it is the closest peer to key 11. At this point, peer 12 can send a message back to the querying peer, peer 3, indicating that it is responsible for key 11.

不仅仅记录前后两个peer, 最优方案是记录logN个peer, 利用shortcut

BitTorrent使用Kademlia DHT

# Socket实现TCP/UDP通讯



### TCP
**UDPClient.py**
```python
from socket import *

serverName = ‘hostname’
serverPort = 12000
clientSocket = socket(socket.AF_INET, socket.SOCK_DGRAM) 
message = raw_input(’Input lowercase sentence:’) 
clientSocket.sendto(message,(serverName, serverPort)) 
modifiedMessage, serverAddress = clientSocket.recvfrom(2048) 
print modifiedMessage
clientSocket.close()
```

**UDPServer.py**
```python
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM) 
serverSocket.bind((’’, serverPort))
print ”The server is ready to receive” 
while 1:
    message, clientAddress = serverSocket.recvfrom(2048) modifiedMessage = message.upper() serverSocket.sendto(modifiedMessage, clientAddress)
```

### UDP
**TCPClient.py**
```python
from socket import *
serverName = ’servername’
serverPort = 12000
clientSocket = socket(AF_INET, SOCK_STREAM) 
clientSocket.connect((serverName,serverPort)) 
sentence = raw_input(‘Input lowercase sentence:’) clientSocket.send(sentence)
modifiedSentence = clientSocket.recv(1024) 
print ‘From Server:’, modifiedSentence clientSocket.close()
```

**TCPServer.py**
```python
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET,SOCK_STREAM) 
serverSocket.bind((‘’,serverPort)) 
serverSocket.listen(1)
print ‘The server is ready to receive’ 
while 1:
    connectionSocket, addr = serverSocket.accept() 
    sentence = connectionSocket.recv(1024) 
    capitalizedSentence = sentence.upper() 
    connectionSocket.send(capitalizedSentence) connectionSocket.close()
```

**connectionSocket, addr = serverSocket.accept()**

When a client knocks on this door, the program invokes the accept() method for serverSocket, which creates a new socket in the server, called connec- tionSocket, dedicated to this particular client. The client and server then complete the handshaking, creating a TCP connection between the client’s clientSocket and the server’s connectionSocket. With the TCP connection established, the client and server can now send bytes to each other over the connection. With TCP, all bytes sent from one side not are not only guaranteed to arrive at the other side but also guaranteed arrive in order.

## Multiplexing and Demultiplexing
我们通过socket发送message, 需要将message 分解然后传给socket, 这叫做multiplexing. The job of gathering data chunks at the source host from different sockets, encapsulating each data chunk with header information (that will later be used in demultiplexing) to create segments, and passing the segments to the network layer is called multiplexing.

把接收到的packet传给server上不同的socket叫Demultiplexing. This job of delivering the data in a transport-layer segment to the correct socket is called demultiplexing.

With port numbers assigned to UDP sockets, we can now precisely describe UDP multiplexing/demultiplexing. Suppose a process in Host A, with UDP port 19157, wants to send a chunk of application data to a process with UDP port 46428 in Host B. The transport layer in Host A creates a transport-layer segment that includes the application data, the source port number (19157 used if the reciever want to reply), the destination port number (46428), and two other values (which will be discussed later, but are unim- portant for the current discussion). The transport layer then passes the resulting seg- ment to the network layer. The network layer encapsulates the segment in an IP datagram and makes a best-effort attempt to deliver the segment to the receiving host. If the segment arrives at the receiving Host B, the transport layer at the receiving host examines the destination port number in the segment (46428) and delivers the segment to its socket identified by port 46428. Note that Host B could be running multiple processes, each with its own UDP socket and associated port number. As UDP segments arrive from the network, Host B directs (demultiplexes) each segment to the appropriate socket by examining the segment’s destination port number.

唯一能够确定一个连接有4个东西：
1. 服务器的IP
2. 服务器的Port
3. 客户端的IP
4. 客户端的Port

5. 只有一个在监听socket，只做监听；
6. 接收到(accept)连接后把该请求分配到线程或子进程(fork)去处理；

# NetWork Layer
The role of the network layer is thus deceptively simple—to move packets from a sending host to a receiving host.

* **Forwarding**. When a packet arrives at a router’s input link, the router must move the packet to the appropriate output link. For example, a packet arriving from Host H1 to Router R1 must be forwarded to the next router on a path to H2. In Section 4.3, we’ll look inside a router and examine how a packet is actually for- warded from an input link to an output link within a router.
* **Routing**. The network layer must determine the route or path taken by packets as they flow from a sender to a receiver. The algorithms that calculate these paths are referred to as routing algorithms. A routing algorithm would determine, for example, the path along which packets flow from H1 to H2.

Forwarding refers to the router-local action of transferring a packet from an input link interface to the appropriate output link interface. Routing refers to the network-wide process that determines the end-to-end paths that packets take from source to destina- tion.

Every router has a **forwarding table**.

[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/route.png)

可以看到table是由routing algorithm 决定的, the routing algorithm determines the values that are inserted into the routers’ forwarding tables.

匹配是根据最长前缀匹配

packet进入到route会被拆开, 根据from address 经过 input port, 通过table得知进入哪个output port, 然后通过Switch fabric分发到对于的output port上, 两个port都会有queue, 如果队列满了, 就会发生packet loss现象.

## Routing Algorithms
The purpose of a routing algorithm is then simple: given a set of routers, with links connecting the routers, a routing algorithm finds a “good” path from source router to destination router.
* **A global routing algorithm** computes the least-cost path between a source and destination using complete, global knowledge about the network.

The Link-State (LS) Routing Algorithm (Dijkstra’s algorithm)
* **In a decentralized routing algorithm**, the calculation of the least-cost path is carried out in an iterative, distributed manner. 

The Distance-Vector (DV) Routing Algorithm (BF算法)

* **In a load-sensitive algorithm**, link costs vary dynami- cally to reflect the current level of congestion in the underlying link. If a high cost is associated with a link that is currently congested, a routing algorithm will tend to choose routes around such a congested link. 

以上算法遇到的问题:
* 每个路由存储大量信息; 迭代的目标太多, 难以converge
* 公司自己想建自己的route algo, 不和外面一致

Both of these problems can be solved by organizing routers into autonomous sys- tems (ASs), with each AS consisting of a group of routers that are typically under the same administrative control (e.g., operated by the same ISP or belonging to the same company network). Routers within the same AS all run the same routing algo- rithm (for example, an LS or DV algorithm) and have information about each other—exactly as was the case in our idealized model in the preceding section. The routing algorithm running within an autonomous system is called an intra- autonomous system routing protocol. It will be necessary, of course, to connect ASs to each other, and thus one or more of the routers in an AS will have the added task of being responsible for forwarding packets to destinations outside the AS; these routers are called **gateway routers**. gateway了解去别的block的path, internal的route只要知道去gateway就行.

## Routing Information Protocol RIP
RIP is a distance-vector protocol. The version of RIP specified in RFC 1058 uses hop count as a cost metric; that is, each link has a cost of 1. The maximum cost of a path is limited to 15, thus limiting the use of RIP to autonomous systems that are fewer than 15 hops in diameter.

### Intra-AS Routing in the Internet: OSPF
At its heart, however, OSPF is a link-state protocol that uses flooding of link-state information and a Dijkstra least-cost path algorithm. With OSPF, a router constructs a complete topological map (that is, a graph) of the entire autonomous system. 

### Inter-AS Routing: BGP
We just learned how ISPs use RIP and OSPF to determine optimal paths for source- destination pairs that are internal to the same AS. Let’s now examine how paths are determined for source-destination pairs that span multiple ASs. The Border Gate- way Protocol version 4. 很复杂

## Broadcast and Multicast Routing
Broadcast是不需要目的地址的
主要还是flooding或者spanning tree算法, 实际用的是flooding算法

## Multicast
We’ve seen in the previous section that with broadcast service, packets are delivered to each and every node in the network. In this section we turn our attention to multicast service, in which a multicast packet is delivered to only a subset of network nodes. A number of emerging network applications require the delivery of packets from one or more senders to a group of receivers. These applications include bulk data transfer (for example, the transfer of a software upgrade from the software developer to users needing the upgrade), streaming continuous media (for example, the transfer of the audio, video, and text of a live lecture to a set of distributed lec- ture participants), shared data applications (for example, a whiteboard or teleconfer- encing application that is shared among many distributed participants), data feeds (for example, stock quotes), Web cache updating, and interactive gaming (for exam- ple, distributed interactive virtual environments or multiplayer games).

Multicast需要多个地址, 所以有一个identity来确定一个group, 每个destination IP就是这个identity里的成员, 至于成员如何加入这个组, 组又是如何管理成员, 则是通过IGMP.

# IPv4
[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/ipv4.png)

# DHCP Dynamic Host Configuration Protocol
For a newly arriving host, the DHCP protocol is a four-step process, as shown in Figure 4.21 for the network setting shown in Figure 4.20. In this figure, yiaddr (as in “your Internet address”) indicates the address being allocated to the newly arriving client. The four steps are:
* **DHCP server discovery**. The first task of a newly arriving host is to find a DHCP server with which to interact. This is done using a DHCP discover message, which a client sends within a UDP packet to port 67. The UDP packet is encap- sulated in an IP datagram. But to whom should this datagram be sent? The host doesn’t even know the IP address of the network to which it is attaching, much less the address of a DHCP server for this network. Given this, the DHCP client creates an IP datagram containing its DHCP discover message along with the broadcast destination IP address of 255.255.255.255 and a “this host” source IP address of 0.0.0.0. The DHCP client passes the IP datagram to the link layer, which then broadcasts this frame to all nodes attached to the subnet.
* **DHCP server offer(s)**. A DHCP server receiving a DHCP discover message responds to the client with a DHCP offer message that is broadcast to all nodes on the subnet, again using the IP broadcast address of 255.255.255.255. (You might want to think about why this server reply must also be broadcast). Since several DHCP servers can be present on the subnet, the client may find itself in the enviable position of being able to choose from among several offers. Each server offer message contains the transaction ID of the received discover mes- sage, the proposed IP address for the client, the network mask, and an IP address lease time—the amount of time for which the IP address will be valid. It is com- mon for the server to set the lease time to several hours or days [Droms 2002].
* **DHCP request**. The newly arriving client will choose from among one or more server offers and respond to its selected offer with a DHCP request message, echoing back the configuration parameters.
* **DHCP ACK**. The server responds to the DHCP request message with a DHCP ACK message, confirming the requested parameters.

# Network Address Translation (NAT)
Suppose a user sitting in a home network behind host 10.0.0.1 requests a Web page on some Web server (port 80) with IP address 128.119.40.186. The host 10.0.0.1 assigns the (arbitrary) source port num- ber 3345 and sends the datagram into the LAN. The NAT router receives the data- gram, generates a new source port number 5001 for the datagram, replaces the source IP address with its WAN-side IP address 138.76.29.7, and replaces the origi- nal source port number 3345 with the new source port number 5001. When generat- ing a new source port number, the NAT router can select any source port number that is not currently in the NAT translation table. (Note that because a port number field is 16 bits long, the NAT protocol can support over 60,000 simultaneous con- nections with a single WAN-side IP address for the router!) NAT in the router also adds an entry to its NAT translation table. The Web server, blissfully unaware that the arriving datagram containing the HTTP request has been manipulated by the NAT router, responds with a datagram whose destination address is the IP address of the NAT router, and whose destination port number is 5001. When this datagram arrives at the NAT router, the router indexes the NAT translation table using the des- tination IP address and destination port number to obtain the appropriate IP address (10.0.0.1) and destination port number (3345) for the browser in the home network. The router then rewrites the datagram’s destination address and destination port number, and forwards the datagram into the home network.

### Universal Plug and Play (UPnP)
解决host behind NAT不能直接和另一个host直接通信的问题
As an example, suppose your host, behind a UPnP-enabled NAT, has private address 10.0.0.1 and is running BitTorrent on port 3345. Also suppose that the public IP address of the NAT is 138.76.29.7. Your BitTorrent application naturally wants to be able to accept connections from other hosts, so that it can trade chunks with them. To this end, the BitTorrent application in your host asks the NAT to cre- ate a “hole” that maps (10.0.0.1, 3345) to (138.76.29.7, 5001). (The public port number 5001 is chosen by the application.) The BitTorrent application in your host could also advertise to its tracker that it is available at (138.76.29.7, 5001). In this manner, an external host running BitTorrent can contact the tracker and learn that your BitTorrent application is running at (138.76.29.7, 5001). The external host can send a TCP SYN packet to (138.76.29.7, 5001). When the NAT receives the SYN packet, it will change the destination IP address and port number in the packet to (10.0.0.1, 3345) and forward the packet through the NAT.

# Internet Control Message Protocol (ICMP)
ICMP is used by hosts and routers to communicate net- work-layer information to each other. The most typical use of ICMP is for error reporting. For example, when running a Telnet, FTP, or HTTP session, you may have encountered an error message such as “Destination network unreachable.” This message had its origins in ICMP. At some point, an IP router was unable to find a path to the host specified in your Telnet, FTP, or HTTP application. That router cre- ated and sent a type-3 ICMP message to your host indicating the error.

ICMP is often considered part of IP but architecturally it lies just above IP, as ICMP messages are carried inside IP datagrams. That is, ICMP messages are carried as IP payload, just as TCP or UDP segments are carried as IP payload. Similarly, when a host receives an IP datagram with ICMP specified as the upper-layer proto- col, it demultiplexes the datagram’s contents to ICMP, just as it would demultiplex a datagram’s content to TCP or UDP.

The well-known ping program sends an ICMP type 8 code 0 message to the specified host. The destination host, seeing the echo request, sends back a type 0 code 0 ICMP echo reply. (ICMP具体数据类型查看ICMP message types)

ICMP也有congestion的作用, 不过是作用在网络层, 不常用

# IPv6
[image](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/IPv6.png)
区别:

* Expanded addressing capabilities. IPv6 increases the size of the IP address from 32 to 128 bits.
* A streamlined 40-byte header. As discussed below, a number of IPv4 fields have been dropped or made optional. The resulting 40-byte fixed-length header allows for faster processing of the IP datagram. A new encoding of options allows for more flexible options processing.
* Flow labeling and priority. For example, audio and video transmission might likely be treated as a flow. On the other hand, the more traditional applications, such as file transfer and e-mail, might not be treated as flows.
* Fragmentation/Reassembly. IPv6 does not allow for fragmentation and reassem- bly at intermediate routers;  
# DNS
DNS is based on UDP
DNS is a distributed database of IP to domain name mappings. Its basic job is to turn a user-friendly domain name like "howstuffworks.com" into an Internet Protocol (IP) address like 70.42.251.42 that computers use to identify each other on the network.

To understand how these three classes of servers interact, suppose a DNS client wants to determine the IP address for the hostname www.amazon.com. To a first approximation, the following events will take place. The client first contacts one of the root servers, which returns IP addresses for TLD servers for the top-level domain com. The client then contacts one of these TLD servers, which returns the IP address of an authoritative server for amazon.com. Finally, the client contacts one of the authoritative servers for amazon.com, which returns the IP address

1. Request information
The process begins when you ask your computer to resolve a hostname, such as visiting http://dyn.com. The first place your computer looks is its local DNS cache, which stores information that your computer has recently retrieved.
If your computer doesn’t already know the answer, it needs to perform a DNS query to find out.
2. Ask the recursive DNS servers
If the information is not stored locally, your computer queries (contacts) your ISP’s recursive DNS servers. These specialized computers perform the legwork of a DNS query on your behalf. Recursive servers have their own caches, so the process usually ends here and the information is returned to the user. TLD的地址也可以被缓存, 因此有可能会跳过第三步
3. Ask the root nameservers
If the recursive servers don’t have the answer, they query the root nameservers. A nameserver is a computer that answers questions about domain names, such as IP addresses. The thirteen root nameservers act as a kind of telephone switchboard for DNS. They don’t know the answer, but they can direct our query to someone that knows where to find it.
4. Ask the TLD nameservers
The root nameservers will look at the first part of our request, reading from right to left — www.dyn.com — and direct our query to the Top-Level Domain (TLD) nameservers for .com. Each TLD, such as .com, .org, and .us, have their own set of nameservers, which act like a receptionist for each TLD. These servers don’t have the information we need, but they can refer us directly to the servers that do have the information.
5. Ask the authoritative DNS servers
The TLD nameservers review the next part of our request — www.dyn.com — and direct our query to the nameservers responsible for this specific domain. These authoritative nameservers are responsible for knowing all the information about a specific domain, which are stored in DNS records. There are many types of records, which each contain a different kind of information. In this example, we want to know the IP address for www.dyndns.com, so we ask the authoritative nameserver for the Address Record (A).
6. Retrieve the record
The recursive server retrieves the A record for dyn.com from the authoritative nameservers and stores the record in its local cache. If anyone else requests the host record for dyn.com, the recursive servers will already have the answer and will not need to go through the lookup process again. All records have a time-to-live value, which is like an expiration date. After a while, the recursive server will need to ask for a new copy of the record to make sure the information doesn’t become out-of-date.
7. Receive the answer
Armed with the answer, recursive server returns the A record back to your computer. Your computer stores the record in its cache, reads the IP address from the record, then passes this information to your browser. The browser then opens a connection to the webserver and receives the website.
This entire process, from start to finish, takes only milliseconds to complete.

### DNS Records and Messages
The DNS servers that together implement the DNS distributed database store resource records (RRs), including RRs that provide hostname-to-IP address map- pings. A resource record is a four-tuple that contains the following fields:

(Name, Value, Type, TTL)

TTL is the time to live of the resource record
# URL vs URI
For starters, URI stands for uniform resource identifier and URL stands for uniform resource locator.

Most of the confusion with these two is because they are related. You see, a URI can be a name, locator, or both for an online resource where a URL is just the locator. URLs are a subset of URIs. 

# Cookie & Session
客户想去银存钱，如果他是第一次去，那么银行就需要给他开户，办一张卡给他，这里银行就类似于server，客户就是client，client获得了银行给他的卡，那么下次他再去的时候，
就不需要重新办卡存钱了，这个卡就是这个人在银行的身份凭证，只要出示这个卡，银行就可以帮他存钱。

同时，银行这里会有这个人的账户，如果这个人存完钱了，还想要继续转钱，通常我们是在柜台（session）进行操作，在出示银行卡之后，柜台会暂时帮你进行操作，这时候客户就可以一直进行操作而不需要每次都出示银行卡之后才能进行操作。

另外，如果这时候这个人离开了，第二天再来重新进行操作，依旧需要重新出示银行卡，因为过了这么久，你和柜台建立的关系已经失效了，这时候需要重新出示银行卡。

## Cookie
当client向server发送请求的时候，server会创建一个cookie，并通过response发送给client，client会保存cookie，下次client发送的request里会附加上这个server的所有cookie,这时候server就会读取cookie里的信息，而不用每次都需要client发送所有信息。

Cookie的缺点：

1. Cookie数量和长度的限制。每个domain最多只能有20条cookie，每个cookie长度不能超过4KB，否则会被截掉。
2. 安全性问题。如果cookie被人拦截了，那人就可以取得所有的session信息。即使加密也与事无补，因为拦截者并不需要知道cookie的意义，他只要原样转发cookie就可以达到目的了。
3. 有些状态不可能保存在客户端。例如，为了防止重复提交表单，我们需要在服务器端保存一个计数器。如果我们把这个计数器保存在客户端，那么它起不到任何作用。

常见的remember me就是用cookie实现的

## Session

当client访问server时，server通过getSession方法创建或获得session，如果创建了一个session，server会同时创建一个jsessionId的cookie，里面包含了这个session的id，下次client发送请求时候，getsession会通过jsessionId获得对应的session。

session会针对不同浏览器持有多份，因此当你打开不同浏览器时，实际上不会保留上一个浏览器的内容，同样，携带不同的cookie也会

Session默认的生命周期是20分钟，可以手动设置更长或更短的时间。

持久化登录用session

## [Encoding vs. Encryption vs. Hashing vs. Obfuscation](https://danielmiessler.com/study/encoding-encryption-hashing-obfuscation/#summary)



# 科普

### P2P

peer to peer, 文件下载, 看视频, 是client和client之间不通过server的交互