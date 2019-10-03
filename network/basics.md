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
The TCP service model includes a connection-oriented service and a reliable data transfer service.

**Connection-oriented service**

TCP has the client and server exchange transport- layer control information with each other before the application-level messages begin to flow. This so-called handshaking procedure alerts the client and server, allowing them to prepare for an onslaught of packets. After the handshaking phase, a TCP connection is said to exist between the sockets of the two processes. The connection is a full-duplex connection in that the two processes can send messages to each other over the connection at the same time. When the application finishes sending messages, it must tear down the connection.

**Reliable data transfer service**

The communicating processes can rely on TCP to deliver all data sent without error and in the proper order. When one side of the application passes a stream of bytes into a socket, it can count on TCP to deliver the same stream of bytes to the receiving socket, with no missing or duplicate bytes.

**congestion-control mechanism**


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