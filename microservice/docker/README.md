# install docker

```
sudo apt install docker.io

docker version

sudo usermod -a -G docker $USER

sudo reboot
```

# mantipulate docker

### show images
```docker images```

### show images' size
```docker system df```

### run docker
```docker run -p 8080:8080 tomcat```

```docker run -p 8080:8080 -d tomcat``` 

### delete docker
```docker rmi```

```docker rm ```

### show running docker
```docker ps```

### show all docker
```docker ps -a```

# MYSQL
**启动数据库**

```docker run -p 3306:3306 --name root -e MYSQL_ROOT_PASSWORD=root -d mysql```

**远程连接数据库**
```mysql -u root -p root -h 192.168.31.130 -P 3306```

# 部署springboot + mysql 服务
[gua](!https://bingohuang.com/spring-boot-docker/)

# tip

VMWare Fusion 改了network之后,虚拟机连不上
```
'sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop' and
'sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start'
```

FTP 无法传输文件问题:

```sudo vi /etc/vsftpd.conf

set write_enable=true

sudo service vsftpd restart
```