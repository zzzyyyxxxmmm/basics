# Configuation
## compile and run nginx
```
1. 进入nginx源码目录
2. ./configure 检查prerequisite
3. make 编译
4. sudo make install 安装到/usr/local/nginx
5. 进入到/usr/local/nginx/sbin
6. 检查配置文件是否正确 sudo ./nginx -t
7. sudo ./nginx 
8. 访问http://localhost/
```

```
1.查看版本号
cd /usr/local/nginx/sbin/
./nginx -v

2. 启动
./nginx

```

# 使用nginx做反向代理
1. 一对一
```
location / {
    root html;
    proxy_pass http://localhost:1323/;
    index index.html index.htm
}
```
2. 根据路径代理到不同的服务器上
```
location ~ /edu {
    proxy_pass http://127.0.0.1:9000;
}

location ~ /vog {
    proxy_pass http://127.0.0.1:9001;
}
```

当客户端发来HTTP请求时，Nginx并不会立刻转发到上游服务器，而是先把用户的请求 (包括HTTP包体)完整地接收到Nginx所在服务器的硬盘或者内存中，然后再向上游服务器 发起连接，把缓存的客户端请求转发到上游服务器。而Squid等代理服务器则采用一边接收客户端请求，一边转发到上游服务器的方式。

Nginx的这种工作方式有什么优缺点呢?很明显，缺点是延长了一个请求的处理时间， 并增加了用于缓存请求内容的内存和磁盘空间。而优点则是降低了上游服务器的负载，尽量 把压力放在Nginx服务器上。

Nginx的这种工作方式为什么会降低上游服务器的负载呢?通常，客户端与代理服务器 之间的网络环境会比较复杂，多半是“走”公网，网速平均下来可能较慢，因此，一个请求可 能要持续很久才能完成。而代理服务器与上游服务器之间一般是“走”内网，或者有专线连 接，传输速度较快。Squid等反向代理服务器在与客户端建立连接且还没有开始接收HTTP包体时，就已经向上游服务器建立了连接。例如，某个请求要上传一个1GB的文件，那么每次 Squid在收到一个TCP分包(如2KB)时，就会即时地向上游服务器转发。在接收客户端完整 HTTP包体的漫长过程中，上游服务器始终要维持这个连接，这直接对上游服务器的并发处 理能力提出了挑战。

Nginx则不然，它在接收到完整的客户端请求(如1GB的文件)后，才会与上游服务器 建立连接转发请求，由于是内网，所以这个转发过程会执行得很快。这样，一个客户端请求 占用上游服务器的连接时间就会非常短，也就是说，Nginx的这种反向代理方案主要是为了 降低上游服务器的并发压力。

## proxy_cache
启用缓存功能, 即使上游服务器断开, nginx也能返回缓存的结果, 但缓存不会更新

# 负载均衡
```
http{
    upstream myserver {
        server 192.168.17.129:8080
        server 192.168.17.129:8081
    }

    server{
        server_name 192.168.17.129

        location / {
            proxy_pass http://myserver;
        }
    }

}
```

# 动静分离
```
data/www/a.html
data/image/1.jpg

server{
    location /www/ {
        root /data/;
        index index.html index htm
    }

    location /image/ {
        root /data/;
        autoindex on;
    }
}
```

# NGINX as a Reverse Proxy
A reverse proxy is a web server that terminates connections with clients and makes new ones to upstream servers on their behalf. An upstream server is defined as a server that NGINX makes a connection with in order to fulfill the client's request. These upstream servers can take various forms, and NGINX can be configured differently to handle each of them.

```
http {
server {
listen       8111;
server_name  localhost;
rewrite ^/(.*)$ http://localhost:1323/$1 permanent;
}
}
```

多路设置:
```
location ~ /edu {
    proxy_pass http://127.0.0.1:9000;
}

location ~ /vog {
    proxy_pass http://127.0.0.1:9001;
}

location /blog {
    proxy_pass http://127.0.0.1:9000;
}

location /tickets {
    proxy_pass http://tickets.example.com; 
}

location /img {
     try_files /static @imageserver;
}

location / {
  root /static;
}

location @imageserver {
    proxy_pass http://127.0.0.1:8080; 
}
```

多个server name:
```
server {
    server_name marketing.example.com marketing.example.org marketing. example.net;
    rewrite ^ http://www.example.com/marketing/application.do permanent; 
}

server {
    server_name communication.example.com communication.example.org communication.example.net;
    rewrite ^ http://www.example.com/comms/index.cgi permanent; 
}

server {
    server_name www.example.org www.example.net;
    rewrite ^ http://www.example.com$request_uri permanent; 
}
```

### error page
```
server {
error_page 500 502 503 504 /50x.html; 
location = /50x.html {
       root share/examples/nginx/html;
     }
}

server {
error_page 500 http://www.example.com/maintenance.html;
}

```

# Reverse Proxy Advanced Topics
As we saw in the previous chapter, a reverse proxy makes connections to upstream servers on behalf of clients. These upstream servers therefore have no direct connection to the client. This is for several different reasons, such as security, scalability,
and performance.

A reverse proxy server aids security because if an attacker were to try to get onto the upstream server directly, he would have to first find a way to get onto the reverse proxy. Connections to the client can be encrypted by running them over HTTPS. These SSL connections may be terminated on the reverse proxy, when the upstream server cannot or should not provide this functionality itself. NGINX can act as an SSL terminator as well as provide additional access lists and restrictions based on various client attributes.

Scalability can be achieved by utilizing a reverse proxy to make parallel connections to multiple upstream servers, enabling them to act as if they were one. If the application requires more processing power, additional upstream servers can be added to the pool served by a single reverse proxy.

Performance of an application may be enhanced through the use of a reverse proxy in several ways. The reverse proxy can cache and compress content before delivering it out to the client. NGINX as a reverse proxy can handle more concurrent client connections than a typical application server. Certain architectures configure NGINX to serve static content from a local disk cache, passing only dynamic requests to
the upstream server to handle. Clients can keep their connections to NGINX alive, while NGINX terminates the ones to the upstream servers immediately, thus freeing resources on those upstream servers.


## HTTP block
* http: This block is inserted at the root of the configuration file. It allows you to start defining directives and blocks from all modules related to the HTTP facet of Nginx. Although there is no real purpose in doing so, the block can be inserted multiple times, in which case the directive values inserted in the last block will override the previous ones.
* server: This block allows you to declare a website. In other words, a specific website (identified by one or more hostnames, for example, www.mywebsite. com) becomes acknowledged by Nginx and receives its own configuration. This block can only be used within the http block.
* location: Lets you define a group of settings to be applied to a particular location on a website. The next part of this section provides more details about the location block. This block can be used within a server block or nested within another location block.

<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/nginx_1.png" width="700" height="500">
</div>

```
http {
# Enable gzip compression at the http block level gzip on;
    server {
        server_name localhost;
        listen 80;
        # At this stage, gzip still set to on
        location /downloads/ { 
            gzip off;
            #    This directive only applies to documents found
            #    in /downloads/
    } 
    }
}
```

**server_name**: Assigns one or more hostnames to the server block. When Nginx receives an HTTP request, it matches the Host header of the request against all of the server blocks. The first server block to match this hostname is selected. 

Plan B: If no server block matches the desired host, Nginx selects the first server block that matches the parameters of the listen directive (such as listen *:80 would be a catch-all for all requests received on port 80), giving priority to the first block that has the default option enabled on the listen directive.


# Nginx process architecture
At the very moment of starting Nginx, one unique process exists in memory—the Master Process. It is launched with the current user and group permissions—usually root/root if the service is launched at boot time by an init script. The master process itself does not process any client request, instead, it spawns processes that do—the Worker Processes, which are affected to a customizable user and group.

nginx采用的是多进程的架构, 相比于多线程, 进程的崩溃不会导致nginx整体的崩溃.

Master fork出worker, cache manager, cache loader, 这些进程都是通过共享内存实现通信的, worker最好绑定在固定的CPU上, 从而增加缓存的命中率

## nginx 信号
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/nginx_2.png" width="700" height="500">
</div> 

## 热升级流程
1. 将旧nginx文件换成新Nginx文件(注意备份)
2. 向master进程发送USR2信号
3. master进程修改pid文件名, 加后缀.oldbin
4. 老master进程用新nginx文件启动新master进程
5. 向老master进程发送QUIT信号, 关闭老master进程(optional)
6. 回滚: 向老master发送HUP, 向新master发送QUIT

## 优雅关闭worker
主要针对http请求. 
1. 设置定时器 worker_shutdown_timeout
2. 关闭监听句柄
3. 关闭空闲连接
4. 在循环中等待全部连接关闭, 耗时
5. 退出进程

## nginx事件处理
<div align=center>
<img src="https://github.com/zzzyyyxxxmmm/basics/blob/master/image/nginx_3.png" width="700" height="500">
</div> 

nginx使用队列保证某个module不会长时间占用CPU

nginx使用epoll处理, 只用处理活跃的连接

## Slab内存管理
将内存分为多个slot, 大小为8,16,32,64, 有多个, 每次分配的时候选一个最小的
* 最多两倍内存消耗,例如64分配了33
* 适合小对象
* 避免碎片
* 避免重复初始化