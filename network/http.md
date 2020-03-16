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



# What Real Web Server Do

1. Set up connection—accept a client connection, or close if the client is unwanted. 2. Receive request—read an HTTP request message from the network.
3. Process request—interpret the request message and take action.
4. Access resource—access the resource specified in the message.
5. Construct response—create the HTTP response message with the right headers. 6. Send response—send the response back to the client.
7. Log transaction—place notes about the completed transaction in a log file.

## Step 1: Accepting Client Connections

## Step 2: Receiving Request Messages

### Connection Input/Output Processing Architectures
Web servers constantly watch for new web requests, because requests can arrive at any time. Different web server architectures service requests in different ways, as Figure 5-7 illustrates:

* Single-threaded web servers (Figure 5-7a)
Single-threaded web servers process one request at a time until completion. When the transaction is complete, the next connection is processed. This archi- tecture is simple to implement, but during processing, all the other connections are ignored. This creates serious performance problems and is appropriate only for low-load servers and diagnostic tools like type-o-serve.
* Multiprocess and multithreaded web servers (Figure 5-7b)

Multiprocess and multithreaded web servers dedicate multiple processes or higher-efficiency threads to process requests simultaneously.* The threads/ processes may be created on demand or in advance.† Some servers dedicate a thread/process for every connection, but when a server processes hundreds, thousands, or even tens of thousands of simultaneous connections, the resulting number of processes or threads may consume too much memory or system
resources. Thus, many multithreaded web servers put a limit on the maximum number of threads/processes.
* Multiplexed I/O servers (Figure 5-7c)

To support large numbers of connections, many web servers adopt multiplexed architectures. In a multiplexed architecture, all the connections are simulta- neously watched for activity. When a connection changes state (e.g., when data becomes available or an error condition occurs), a small amount of processing is performed on the connection; when that processing is complete, the connection is returned to the open connection list for the next change in state. Work is done on a connection only when there is something to be done; threads and processes are not tied up waiting on idle connections.
* Multiplexed multithreaded web servers (Figure 5-7d)

Some systems combine multithreading and multiplexing to take advantage of multiple CPUs in the computer platform. Multiple threads (often one per physi- cal processor) each watch the open connections (or a subset of the open connec- tions) and perform a small amount of work on each connection.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_1.png" width="500" height="500">
</div>

## Step 3: Processing Requests

## Step 4: Mapping and Accessing Resources

### Docroots
Web servers support different kinds of resource mapping, but the simplest form of resource mapping uses the request URI to name a file in the web server’s filesystem. Typically, a special folder in the web server filesystem is reserved for web content. This folder is called the document root, or docroot. The web server takes the URI from the request message and appends it to the document root.

In Figure 5-8, a request arrives for /specials/saw-blade.gif. The web server in this example has document root /usr/local/httpd/files. The web server returns the file /usr/ local/httpd/files/specials/saw-blade.gif.

To set the document root for an Apache web server, add a DocumentRoot line to the httpd.conf configuration file:

**DocumentRoot /usr/local/httpd/files**

Servers are careful not to let relative URLs back up out of a docroot and expose other parts of the filesystem. For example, most mature web servers will not permit this URI to see files above the Joe’s Hardware document root:
**http://www.joes-hardware.com/../**

### Virtually hosted docroots
不同的Host映射到不同的文件夹里
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_2.png" width="500" height="300">
</div>

### User home directory docroots
不同的路径映射到不同的文件夹里
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_3.png" width="500" height="300">
</div>

### Directory Listings
A web server can receive requests for directory URLs, where the path resolves to a directory, not a file. Most web servers can be configured to take a few different actions when a client requests a directory URL:
* Return an error.
* Return a special, default, “index file” instead of the directory.
* Scan the directory, and return an HTML page containing the contents.

Most web servers look for a file named index.html or index.htm inside a directory to represent that directory. If a user requests a URL for a directory and the directory contains a file named index.html (or index.htm), the server will return the contents of that file.
In the Apache web server, you can configure the set of filenames that will be inter- preted as default directory files using the DirectoryIndex configuration directive. The DirectoryIndex directive lists all filenames that serve as directory index files, in pre- ferred order. The following configuration line causes Apache to search a directory for any of the listed files in response to a directory URL request:

DirectoryIndex index.html index.htm home.html home.htm index.cgi
     
If no default index file is present when a user requests a directory URI, and if direc- tory indexes are not disabled, many web servers automatically return an HTML file listing the files in that directory, and the sizes and modification dates of each file, including URI links to each file. This file listing can be convenient, but it also allows nosy people to find files on a web server that they might not normally find.

## Step 5: Building Responses
## Step 6: Sending Responses
## Step 7: Logging

# Proxies

## Proxies Versus Gateways
Strictly speaking, proxies connect two or more applications that speak the same pro- tocol, while gateways hook up two or more parties that speak different protocols. A gateway acts as a “protocol converter,” allowing a client to complete a transaction with a server, even when the client and server speak different protocols.


# Cache
1. 浏览器先检查是否有cache可用, 如果有并且有效则直接返回缓存
2. 经过缓存, 缓存判断是否有效, 先看expire date, 如果没过期直接返回, 如果过期则发送一个conditional request向服务器询问是否更新, 同时更新缓存expire date
3. 返回结果

client first sends a request, it will first check whether the local storage (local storage is established by local ISP)has cache or not. If yes, it just return the cache in response. If not, it will send the request to the server, and when response return, it will be cached for the next time request. The response has a header called Last-Modified which indicate the last time the object was modified.

What if the content they cache is modified on server? The cache send a request with a header called If-modified-since. The request will go to the server and check if the Last-Modified is same with If-modified-since. If yes, it will return 304 to tell the client to get the cache directly. If not, it will return a normall response and the response will be cached again. WIthout fetching the entire object from the server.

## Cache Processing Steps
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_4.png" width="500" height="500">
</div>

Modern commercial proxy caches are quite complicated. They are built to be very high-performance and to support advanced features of HTTP and other technologies. But, despite some subtle details, the basic workings of a web cache are mostly simple. A basic cache-processing sequence for an HTTP GET message consists of seven steps (illustrated in Figure 7-11):
1. Receiving—Cache reads the arriving request message from the network. 
2. Parsing—Cache parses the message, extracting the URL and headers.
3. Lookup—Cache checks if a local copy is available and, if not, fetches a copy (and stores it locally).
4. Freshness check—Cache checks if cached copy is fresh enough and, if not, asks server for any updates.
5. Response creation—Cache makes a response message with the new headers and cached body.
6. Sending—Cache sends the response back to the client over the network.
7. Logging—Optionally, cache creates a log file entry describing the transaction.

## Cache Header

### Expiration Dates and Ages
**Cache-Control: max-age**

The max-age value defines the maximum age of the document—the maximum legal elapsed time (in seconds) from when a document is first generated to when it can no longer be considered fresh enough to serve.

*Cache-Control: max-age=484200*

**Expires**
Specifies an absolute expiration date. If the expiration date is in the past, the document is no longer fresh.

*Expires: Fri, 05 Jul 2002, 05:00:00 GMT*

### conditional request
**If-Modified-Since:<date>** 

Perform the requested method if the document has been modified since the specified date. This is used in conjunction with the Last-Modified server response header, to fetch content only if the content has been modified from the cached version.

**If-None-Match: <tags>**

Instead of matching on last-modified date, the server may provide special tags (see “ETag” in Appendix C) on the document that act like serial numbers. The If-None-Match header performs the requested method if the cached tags differ from the tags in the server’s document.

### Cache Control
* Attach a Cache-Control: no-store header to the response.
* Attach a Cache-Control: no-cache header to the response.
* Attach a Cache-Control: must-revalidate header to the response.
* Attach a Cache-Control: max-age header to the response.
* Attach an Expires date header to the response.
* Attach no expiration information, letting the cache determine its own heuristic expiration date.

# Integration Points: Gateways, Tunnels, and Relays

### What is gateway
The history behind HTTP extensions and interfaces was driven by people’s needs. When the desire to put more complicated resources on the Web emerged, it rapidly became clear that no single application could handle all imaginable resources.

To get around this problem, developers came up with the notion of a gateway that could serve as a sort of interpreter, abstracting a way to get at the resource. A gate- way is the glue between resources and applications. An application can ask (through HTTP or some other defined interface) a gateway to handle the request, and the gateway can provide a response. The gateway can speak the query language to the database or generate the dynamic content, acting like a portal: a request goes in, and a response comes out.

Some gateways automatically translate HTTP traffic to other protocols, so HTTP cli- ents can interface with other applications without the clients needing to know other protocols.

Figure 8-2 shows three examples of gateways:
* In Figure 8-2a, the gateway receives HTTP requests for FTP URLs. The gateway then opens FTP connections and issues the appropriate commands to the FTP server. The document is sent back through HTTP, along with the correct HTTP headers.
* In Figure 8-2b, the gateway receives an encrypted web request through SSL, decrypts the request,* and forwards a normal HTTP request to the destination server. These security accelerators can be placed directly in front of web servers (usually in the same premises) to provide high-performance encryption for ori- gin servers.

## Tunnels
We’ve discussed different ways that HTTP can be used to enable access to various kinds of resources (through gateways) and to enable application-to-application com- munication. In this section, we’ll take a look at another use of HTTP, web tunnels, which enable access to applications that speak non-HTTP protocols through HTTP applications.

Web tunnels let you send non-HTTP traffic through HTTP connections, allowing other protocols to piggyback on top of HTTP. The most common reason to use web tunnels is to embed non-HTTP traffic inside an HTTP connection, so it can be sent through firewalls that allow only web traffic.

Figure 8-10 shows how the CONNECT method works to establish a tunnel to a gateway:
* In Figure 8-10a, the client sends a CONNECT request to the tunnel gateway. The client’s CONNECT method asks the tunnel gateway to open a TCP connec- tion (here, to the host named orders.joes-hardware.com on port 443, the normal SSL port).
* The TCP connection is created in Figure 8-10b and Figure 8-10c.
* Once the TCP connection is established, the gateway notifies the client (Figure 8-10d) by sending an HTTP 200 Connection Established response.
* At this point, the tunnel is set up. Any data sent by the client over the HTTP tunnel will be relayed directly to the outgoing TCP connection, and any data sent by the server will be relayed to the client over the HTTP tunnel.
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_5.png" width="700" height="500">
</div>


### Establishing HTTP Tunnels with CONNECT
The CONNECT method asks a tunnel gateway to create a TCP connection to an arbitrary destination server and port and to blindly relay subsequent data between client and server.

### SSL Tunnnel
Web tunnels were first developed to carry encrypted SSL traffic through firewalls. Many organizations funnel all traffic through packet-filtering routers and proxy serv- ers to enhance security. But some protocols, such as encrypted SSL, cannot be prox- ied by traditional proxy servers, because the information is encrypted. Tunnels let the SSL traffic be carried through the port 80 HTTP firewall by transporting it through an HTTP connection.

To allow SSL traffic to flow through existing proxy firewalls, a tunneling feature was added to HTTP, in which raw, encrypted data is placed inside HTTP messages and sent through normal HTTP channels.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_6.png" width="700" height="500">
</div>

In Figure 8-12a, SSL traffic is sent directly to a secure web server (on SSL port 443). In Figure 8-12b, SSL traffic is encapsulated into HTTP messages and sent over HTTP port 80 connections, until it is decapsulated back into normal SSL connections.

Tunnels often are used to let non-HTTP traffic pass through port-filtering firewalls. This can be put to good use, for example, to allow secure SSL traffic to flow through firewalls. However, this feature can be abused, allowing malicious protocols to flow into an organization through the HTTP tunnel.

# Secure HTTP
HTTPS 相当于在HTTP和TCP中间加了一层SSL

## Digital Cryptography
Before we talk in detail about HTTPS, we need to provide a little background about the cryptographic encoding techniques used by SSL and HTTPS. In the next few sec- tions, we’ll give a speedy primer of the essentials of digital cryptography. If you already are familiar with the technology and terminology of digital cryptography, feel free to jump ahead to “HTTPS: The Details.”
In this digital cryptography primer, we’ll talk about:

* **Ciphers** Algorithms for encoding text to make it unreadable to voyeurs
* **Keys** Numeric parameters that change the behavior of ciphers
* **Symmetric-key** cryptosystems Algorithms that use the same key for encoding and decoding
* **Asymmetric-key** cryptosystems Algorithms that use different keys for encoding and decoding
* **Public-key** cryptography A system making it easy for millions of computers to send secret messages
* **Digital signatures** Checksums that verify that a message has not been forged or tampered with
* **Digital certificates** Identifying information, verified and signed by a trusted organization

## Digital Signatures
好比考驾照, 政府给我一个驾照和我自己给自己一个驾照是不一样的, 那么政府怎么知道这个驾照是不是自己伪造的呢. 在颁发驾照的之后zf通过融合各种信息生成一个digest并且用private key加密, 放在驾照上. 别的机构想要验证就可以通过public key解密, 查看这个digest是否正确, 而我个人因为没有private key, 因此加密过的信息无法通过public key解开, 因此无法验证. 这个加密过后的digest就相当于签名

## Digital Certificates
这个就相当于驾照

## Secure Transport Setup
The procedure is slightly more complicated in HTTPS, because of the SSL security layer. In HTTPS, the client first opens a connection to port 443 (the default port for secure HTTP) on the web server. Once the TCP connection is established, the client and server initialize the SSL layer, negotiating cryptography parameters and exchang- ing keys. When the handshake completes, the SSL initialization is done, and the cli- ent can send request messages to the security layer. These messages are encrypted before being sent to TCP. 

## Tunneling Secure Traffic Through Proxies
But once the client starts encrypting the data to the server, using the server’s public key, the proxy no longer has the ability to read the HTTP header! And if the proxy can- not read the HTTP header, it won’t know where to forward the request.

To make HTTPS work with proxies, a few modifications are needed to tell the proxy where to connect. One popular technique is the HTTPS SSL tunneling protocol. Using the HTTPS tunneling protocol, the client first tells the proxy the secure host and port to which it wants to connect. It does this in plaintext, before encryption starts, so the proxy can read this information.
HTTP is used to send the plaintext endpoint information, using a new extension method called CONNECT. The CONNECT method tells the proxy to open a con- nection to the desired host and port number and, when that’s done, to tunnel data directly between the client and server. The CONNECT method is a one-line text command that provides the hostname and port of the secure origin server, separated by a colon. The host:port is followed by a space and an HTTP version string fol- lowed by a CRLF. After that there is a series of zero or more HTTP request header lines, followed by an empty line. After the empty line, if the handshake to establish the connection was successful, SSL data transfer can begin.

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


[https建立过程](https://github.com/zzzyyyxxxmmm/basics/blob/master/image/https.png)