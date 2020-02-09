# URL vs URI
A URI can be a name, locator, or both for an online resource where a URL is just the locator. URLs are a subset of URIs. 

Say you want to fetch the URL http://www.joes-hardware.com/seasonal/index-fall.html: • ThefirstpartoftheURL(http)istheURLscheme.Theschemetellsawebclient
how to access the resource. In this case, the URL says to use the HTTP protocol.
* The second part of the URL (www.joes-hardware.com) is the server location.
This tells the web client where the resource is hosted.
* The third part of the URL (/seasonal/index-fall.html) is the resource path. The
path tells what particular local resource on the server is being requested.

URLs provide a way to uniformly name resources. Most URLs have the same “**scheme://server location/path**” structure. 

Most URL schemes base their URL syntax on this nine-part general format:
```html
<scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>
```

| Component | Description                                                                                                                                                                                                                                                                                                                                                                                          | Default value   |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|
| scheme    | Which protocol to use when accessing a server to get a resource.                                                                                                                                                                                                                                                                                                                                     | None            |
| user      | The username some schemes require to access a resource.                                                                                                                                                                                                                                                                                                                                              | anonymous       |
| password  | The password that may be included after the username, separated by a colon (:).                                                                                                                                                                                                                                                                                                                      | <Email address> |
| host      | The hostname or dotted IP address of the server hosting the resource.                                                                                                                                                                                                                                                                                                                                | None            |
| port      | The port number on which the server hosting the resource is listening. Many schemes have default port numbers (the default port number for HTTP is 80).                                                                                                                                                                                                                                              | Scheme-specific |
| path      | The local name for the resource on the server, separated from the previous URL com- ponents by a slash (/). The syntax of the path component is server and scheme specific. (We will see later in this chapter that a URL’s path can be divided into segments, and each segment can have its own components specific to that segment.)                                                            | None            |
| params    | Used by some schemes to specify input parameters. Params are name/value pairs. AURL can contain multiple params fields,  separated from themselves and the rest of the path by semicolons (;). For example, take a protocol like FTP, which has two modes of transfer, binary and text. You wouldn’t want your binary image transferred in text mode, because the binary image could be scrambled. | None            |
| query     | Used by some schemes to pass parameters to active applications (such as databases, bulletin boards, search engines, and other Internet gateways). There is no common format for the contents of the query component. It is separated from the rest of the URL by the “?” character.                                                                                                                  | None            |
| frag      | A name for a piece or part of the resource. The frag field is not passed to the server when referencing the object; it is used internally by the client. It is separated from the rest of the URL by the “#” character.                                                                                                                                                                              | None            |

# HTTP
The HyperText Transfer Protocol (HTTP), the Web’s application-layer protocol, is at the heart of the Web. HTTP is implemented in two programs: a client program and a server program. The client program and server program, executing on different end systems, talk to each other by exchanging HTTP messages. HTTP defines the structure of these messages and how the client and server exchange the messages.

It is important to note that the server sends requested files to clients without stor- ing any state information about the client. If a particular client asks for the same object twice in a period of a few seconds, the server does not respond by saying that it just served the object to the client; instead, the server resends the object, as it has com- pletely forgotten what it did earlier. Because an HTTP server maintains no informa- tion about the clients, HTTP is said to be a **stateless protocol**.

### request 10 images
normally build 5-10 parallel TCP connections

## HTTP Request Message
```
<method> <request-URL> <version>
<headers>
<entity-body>

example:

GET /somedir/page.html HTTP/1.1             --request line (method, URL, HTTP version)
Host: www.someschool.edu                    --request header
Connection: close 
User-agent: Mozilla/5.0 
Accept-language: fr
                                            -- blank line
                                            --request body
```

### Method
The HEAD method is similar to the GET method. When a server receives a request with the HEAD method, it responds with an HTTP message but it leaves out the requested object. Application developers often use the HEAD method for debug- ging. 

The PUT method is often used in conjunction with Web publishing tools. It allows a user to upload an object to a specific path (directory) on a specific Web server. The PUT method is also used by applications that need to upload objects to Web servers.

The POST method was designed to send input data to the server.* In practice, it is often used to support HTML forms. The data from a filled-in form typically is sent to the server, which then marshals it off to where it needs to go (e.g., to a server gateway program, which then processes it). Figure 3-10 shows a client making an HTTP request—sending form data to a server—with the POST method.

The DELETE method allows a user, or an application, to delete an object on a Web server.

The OPTIONS method asks the server to tell us about the various supported capabil- ities of the web server. You can ask a server about what methods it supports in gen- eral or for particular resources. (Some servers may support particular operations only on particular kinds of objects). This provides a means for client applications to determine how best to access vari- ousresourceswithoutactuallyhavingtoaccessthem.

### Header
**Accept headers**

Accept headers give the client a way to tell servers their preferences and capabilities: what they want, what they can use, and, most importantly, what they don’t want. Servers can then use this extra information to make more intelligent decisions about what to send.

## HTTP Response Message
```
<version> <status> <reason-phrase>
<headers>
<entity-body>

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

# TCP and HTTP

### http message
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_message.png" width="700">
</div>

### implement http by socket
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_socket.png" width="700" height="500">
</div>

## TCP Performance Considerations

### HTTP Transaction Delays
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_delay.png" width="700" height="300">
</div>

Notice that the transaction processing time can be quite small compared to the time required to set up TCP connections and transfer the request and response messages. Unless the client or server is overloaded or executing complex dynamic resources, most HTTP delays are caused by TCP network delays.

### TCP Connection Handshake Delays
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_handshake_delay.png" width="700" height="300">
</div>

step c can carry data.

### TCP Slow Start
Because of this congestion-control feature, new connections are slower than “tuned” connections that already have exchanged a modest amount of data. Because tuned connections are faster, HTTP includes facilities that let you reuse existing connec- tions. We’ll talk about these HTTP “persistent connections” later in this chapter.

### Nagle’s Algorithm and TCP_NODELAY
TCP has a data stream interface that permits applications to stream data of any size to the TCP stack—even a single byte at a time! But because each TCP segment car- ries at least 40 bytes of flags and headers, network performance can be degraded severely if TCP sends large numbers of packets containing small amounts of data.*

Nagle’s algorithm (named for its creator, John Nagle) attempts to bundle up a large amount of TCP data before sending a packet, aiding network efficiency. The algo- rithm is described in RFC 896, “Congestion Control in IP/TCP Internetworks.”

Nagle’s algorithm discourages the sending of segments that are not full-size (a maximum-size packet is around 1,500 bytes on a LAN, or a few hundred bytes across the Internet). Nagle’s algorithm lets you send a non-full-size packet only if all other packets have been acknowledged. If other packets are still in flight, the partial data is buffered. This buffered data is sent only when pending packets are acknowl- edged or when the buffer has accumulated enough data to send a full packet.

Nagle’s algorithm causes several HTTP performance problems. First, small HTTP messages may not fill a packet, so they may be delayed waiting for additional data that will never arrive. Second, Nagle’s algorithm interacts poorly with disabled acknowledgments—Nagle’s algorithm will hold up the sending of data until an acknowledgment arrives, but the acknowledgment itself will be delayed 100–200 milliseconds by the delayed acknowledgment algorithm.

HTTP applications often disable Nagle’s algorithm to improve performance, by setting the TCP_NODELAY parameter on their stacks. If you do this, you must ensure that you write large chunks of data to TCP so you don’t create a flurry of small packets.

### TIME_WAIT Accumulation and Port Exhaustion
TIME_WAIT port exhaustion is a serious performance problem that affects perfor- mance benchmarking but is relatively uncommon in real deployments. It warrants special attention because most people involved in performance benchmarking even- tually run into this problem and get unexpectedly poor performance.

When a TCP endpoint closes a TCP connection, it maintains in memory a small con- trol block recording the IP addresses and port numbers of the recently closed con- nection. This information is maintained for a short time, typically around twice the estimated maximum segment lifetime (called “2MSL”; often two minutes*), to make sure a new TCP connection with the same addresses and port numbers is not cre- ated during this time. This prevents any stray duplicate packets from the previous connection from accidentally being injected into a new connection that has the same addresses and port numbers. In practice, this algorithm prevents two connections with the exact same IP addresses and port numbers from being created, closed, and recreated within two minutes.

Today’s higher-speed routers make it extremely unlikely that a duplicate packet will show up on a server’s doorstep minutes after a connection closes. Some operating systems set 2MSL to a smaller value, but be careful about overriding this value. Pack- ets do get duplicated, and TCP data will be corrupted if a duplicate packet from a past connection gets inserted into a new stream with the same connection values.

The 2MSL connection close delay normally is not a problem, but in benchmarking situations, it can be. It’s common that only one or a few test load-generation com- puters are connecting to a system under benchmark test, which limits the number of client IP addresses that connect to the server. Furthermore, the server typically is lis- tening on HTTP’s default TCP port, 80. These circumstances limit the available combinations of connection values, at a time when port numbers are blocked from reuse by TIME_WAIT.

In a pathological situation with one client and one web server, of the four values that make up a TCP connection:

```
<source-IP-address, source-port, destination-IP-address, destination-port>
```

three of them are fixed—only the source port is free to change:

```
<client-IP, source-port, server-IP, 80>
```

Each time the client connects to the server, it gets a new source port in order to have a unique connection. But because a limited number of source ports are available (say, 60,000) and no connection can be reused for 2MSL seconds (say, 120 sec- onds), this limits the connect rate to 60,000 / 120 = 500 transactions/sec. If you keep making optimizations, and your server doesn’t get faster than about 500 transac- tions/sec, make sure you are not experiencing TIME_WAIT port exhaustion. You can fix this problem by using more client load-generator machines or making sure the client and server rotate through several virtual IP addresses to add more connec- tion combinations.

Even if you do not suffer port exhaustion problems, be careful about having large numbers of open connections or large numbers of control blocks allocated for con- nection in wait states. Some operating systems slow down dramatically when there are numerous open connections or control blocks.

## Keep-Alive and Dumb Proxies

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_dumb_proxy.png" width="700" height="300">
</div>

The problem comes with proxies—in particular, proxies that don’t understand the Connection header and don’t know that they need to remove the header before proxy- ing it down the chain. Many older or simple proxies act as blind relays, tunneling bytes from one connection to another, without specially processing the Connection header.

Imagine a web client talking to a web server through a dumb proxy that is acting as a blind relay. This situation is depicted in Figure 4-15.
Here’s what’s going on in this figure:

1. In Figure 4-15a, a web client sends a message to the proxy, including the Connec- tion: Keep-Alive header, requesting a keep-alive connection if possible. The client waits for a response to learn if its request for a keep-alive channel was granted.
2. The dumb proxy gets the HTTP request, but it doesn’t understand the Connec- tion header (it just treats it as an extension header). The proxy has no idea what keep-alive is, so it passes the message verbatim down the chain to the server (Figure 4-15b). But the Connection header is a hop-by-hop header; it applies to only a single transport link and shouldn’t be passed down the chain. Bad things are about to happen.
3. In Figure 4-15b, the relayed HTTP request arrives at the web server. When the web server receives the proxied Connection: Keep-Alive header, it mistakenly concludes that the proxy (which looks like any other client to the server) wants to speak keep-alive! That’s fine with the web server—it agrees to speak keep- alive and sends a Connection: Keep-Alive response header back in Figure 4-15c. So, at this point, the web server thinks it is speaking keep-alive with the proxy and will adhere to rules of keep-alive. But the proxy doesn’t know the first thing about keep-alive. Uh-oh.
4. In Figure 4-15d, the dumb proxy relays the web server’s response message back to the client, passing along the Connection: Keep-Alive header from the web server. The client sees this header and assumes the proxy has agreed to speak keep-alive. So at this point, both the client and server believe they are speaking keep-alive, but the proxy they are talking to doesn’t know anything about keep-alive.
5. Because the proxy doesn’t know anything about keep-alive, it reflects all the data it receives back to the client and then waits for the origin server to close the connection. But the origin server will not close the connection, because it believes the proxy explicitly asked the server to keep the connection open. So the proxy will hang waiting for the connection to close.
6. When the client gets the response message back in Figure 4-15d, it moves right along to the next request, sending another request to the proxy on the keep-alive connection (see Figure 4-15e). Because the proxy never expects another requeston the same connection, the request is ignored and the browser just spins, mak- ing no progress.
7. This miscommunication causes the browser to hang until the client or server times out the connection and closes it.*

To avoid this kind of proxy miscommunication, modern proxies must never proxy the Connection header or any headers whose names appear inside the Connection values. So if a proxy receives a Connection: Keep-Alive header, it shouldn’t proxy either the Connection header or any headers named Keep-Alive.

In addition, there are a few hop-by-hop headers that might not be listed as values of a Connection header, but must not be proxied or served as a cache response either. These include Proxy-Authenticate, Proxy-Connection, Transfer-Encoding, and Upgrade. For more information, refer back to “The Oft-Misunderstood Connection Header.”

解决的方式是通过smart proxy和Proxy-Connection: Keep-Alive

## Connections
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_connections.png" width="700" height="1200">
</div>
## Cache

client first sends a request, it will first check whether the local storage (local storage is established by local ISP)has cache or not. If yes, it just return the cache in response. If not, it will send the request to the server, and when response return, it will be cached for the next time request. The response has a header called Last-Modified which indicate the last time the object was modified.

What if the content they cache is modified on server? The send a request with a header called If-modified-since. The request will go to the server and check if the Last-Modified is same with If-modified-since. If yes, it will return 304 to tell the client to get the cache directly. If not, it will return a normall response and the response will be cached again.

[https建立过程](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/https.png)




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
To get around the limitations of a safe character set representation, an encoding scheme was devised to represent characters in a URL that are not safe. The encoding simply represents the unsafe character by an “escape” notation, consisting of a per- cent sign (%) followed by two hexadecimal digits that represent the ASCII code of the character.
