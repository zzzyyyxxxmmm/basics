# docker 运行jenkins

```docker pull jenkinsci/blueocean```

jenkins的容器启动后，重要的文件我们希望能保存在当前电脑，否则容器被损坏或者删除后就找不回这些文件了，因此要在当前电脑上准备一个目录作为文件映射，注意文件夹权限问题，我这边准备的本机目录是/usr/local/work/jenkins，并且执行了chmod 777 /usr/local/work/jenkins以确保docker进程有权限读写此目录；

```mkdir /usr/local/work/jenkins```
```chmod 777 /usr/local/work/jenkins```
```docker run -p 8080:8080 -p 50000:50000 -v /usr/local/work/jenkins:/var/jenkins_home --name j01 -idt jenkinsci/blueocean```