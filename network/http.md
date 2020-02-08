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

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/http_message.png" width="900">
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
