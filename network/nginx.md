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

## Nginx process architecture
At the very moment of starting Nginx, one unique process exists in memory—the Master Process. It is launched with the current user and group permissions—usually root/root if the service is launched at boot time by an init script. The master process itself does not process any client request, instead, it spawns processes that do—the Worker Processes, which are affected to a customizable user and group.