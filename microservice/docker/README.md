# install docker

```
sudo apt install docker.io

docker version

sudo usermod -a -G docker $USER

sudo reboot
```

# 一些基础操作

**关闭防火墙:**  ```sudo ufw disable```

**ftp开启写mode:**  ```vi /etc/vsftp.conf   sudo service vsftp restart```

# mantipulate docker

### show images
```docker images```

```docker image ls```

```docker image ls -a``` 查看中间层镜像

### show images' size
```docker system df```

### run docker
当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括： 
1. 检查本地是否存在指定的镜像，不存在就从公有仓库下载 
2. 利用镜像创建并启动一个容器 
3. 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层 
4. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去 
5. 从地址池配置一个 ip 地址给容器 
6. 执行用户指定的应用程序 
7. 执行完毕后容器被终止

```docker run -p 8080:8080 tomcat```

```docker run -p 8080:8080 -d tomcat``` 

```docker logs -f name``` 查看运行容器的log

常用参数:
-it 交互终端, t让Docker分配一个伪终端(pseudo-tty)并绑定到容器的标准输入上, i则让容器的标准输入保持打开

--rm 退出后随即将其删除

### delete docker
```docker rmi```

```docker rm ```

```docker image rm $(docker image ls -q redis)```

```docker container prune``` 删除停止的容器

### show running docker
```docker ps```

### show all docker
```docker ps -a```

### dockerhub
```docker login```如果push自己的image需要登录

```docker search centos``` search相关的image

# Custom Docker Image

## 部署docker私服
```c++
sudo docker pull registry:latest
sudo docker run -d -p 5000:5000 --name server-registry -v /tmp/registry:/tmp/registry docker.io/registry:latest
/*
Create or modify /etc/docker/daemon.json on the client machine

{ "insecure-registries":["myregistry.example.com:5000"] }

Restart docker daemon

sudo /etc/init.d/docker restart
*/
docker tag springdemo 192.168.31.132:5000/springdemo
docker push 192.168.31.132:5000/springdemo
docker pull 192.168.31.132:5000/springdemo
```
## Docker Commit

1. nginx
```docker run --name webserver -d -p 80:80 nginx```
2. 进入nginx修改
```
echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
exit
```
3. 查看diff
```docker diff webserver```
4. commit 
```docker commit --author "Jikang Wang <wjk32111@gmail.com>" --message "add hello docker" webserver nginx:v2```
5. docker history webserver

实际中并不用commit, 首先, 如果仔细观察之前的docker diff webserver的结果, 你会发现除了真正想要修改的index.htm了文件外, 由于命令的执行, 还会有很多文件被改动或添加. 这还仅仅只是最简单的操作, 如果是安装软件包,编译构建, 那还会有大量无关的内容被添加进来,如果不小心清理, 将会导致镜像极为臃肿.

此外,使用commit以为着所有对镜像的操作都是黑箱操作,生成的镜像也被称为黑箱镜像, 换句话说, 就是除了制作镜像的人知道执行过什么命令, 怎么生成的镜像, 别人根本无从得知. 而且, 即使是这个制作镜像的人, 过一段时间也无法记得具体操作.

## Dockerfile
Dockerfile 是一个文本文件, 其内容包含了一条条的指令, 每一条指令构建一层, 因此每一条指令的内容, 就是描述改层应当如何构建.

```
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

FROM scrach, scrach是一个虚拟的概念, 相当于一个空白的镜像

每run一次都会commit一次, 建立一个image, 下一层就在这层的基础上重新建立, 我们可以使用&&来连接命令

```docker build -t nginx:v3 .```

### Context
可以看到上面命令的最后有一个., 这实际就是制定了上下文路径, docker build 命令其实是在docker server 上运行的, 如果想要copy 本机文件到server上, 我们就需要上下文的命令.

如果在Dockerfile中这么写:

```COPY ./package.json /app/```
这个.就是我们命令中的., docker build命令发送的时候会把这个目录页发送到server上, 所以上下文目录不能放太大无关的东西. .dockerignore可以用于忽略某些文件

另外docker还支持:
```
docker build https://github.com/abc/abc.git
docker build https://server/context.tar.gz
```

# Dangling Image
随着官方镜像维护, 发布了新版本后, 重新pull时, 镜像名被转移到了新下载的镜像上, 而旧的镜像上的名称则被取消. Docker build也可能导致这种情况

**查看Dangling image:**```docker image ls -f dangling=true```

**删除:**```docker image prune```

# Volume

```s
# 查看所有volume
docker volume ls
# 删除指定volume
docker volume rm [volume name]
# 查看volume详细
docker volume inspect [volume name]
```


# MYSQL
**启动数据库**```docker run -p 3306:3306 --name root -e MYSQL_ROOT_PASSWORD=root -d mysql```

**启动数据库并指定volume**```docker run -p 3306:3306 -v mysql:/var/lib/mysql --name root -e MYSQL_ROOT_PASSWORD=root -d mysql```

**远程连接数据库**```mysql -u root -p root -h 192.168.31.130 -P 3306 -D mysql```

**进入mysql**```docker exec -it root bash```

# 部署springboot + mysql 服务
[gua](https://bingohuang.com/spring-boot-docker/)

# tip

**VMWare Fusion 改了network之后,虚拟机连不上**
```
'sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop' and
'sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start'
```

**FTP 无法传输文件问题:**

```
sudo vi /etc/vsftpd.conf

set write_enable=true

sudo service vsftpd restart
```