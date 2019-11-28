# NIO
Selector能够检测多个注册的通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的响应处理。这样一来，只是用一个单线程就可以管理多个通道，也就是管理多个连接。这样使得只有在连接真正有读写事件发生时，才会调用函数来进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程，并且避免了多线程之间的上下文切换导致的开销。

# Epoll
其实在Linux下设计并发网络程序，向来不缺少方法，比如典型的Apache模型（Process Per Connection，简称PPC），TPC（Thread PerConnection）模型，以及select模型和poll模型，那为何还要再引入Epoll这个东东呢？那还是有得说说的…

### PPC/TPC模型
这两种模型思想类似，就是让每一个到来的连接一边自己做事去，别再来烦我。只是PPC是为它开了一个进程，而TPC开了一个线程。可是别烦我是有代价的，它要时间和空间啊，连接多了之后，那么多的进程/线程切换，这开销就上来了；因此这类模型能接受的最大连接数都不会高，一般在几百个左右。

### select模型

1. 最大并发数限制，因为一个进程所打开的FD（文件描述符）是有限制的，由FD_SETSIZE设置，默认值是1024/2048，因此Select模型的最大并发数就被相应限制了。自己改改这个FD_SETSIZE？想法虽好，可是先看看下面吧…
2. 效率问题，select每次调用都会线性扫描全部的FD集合，这样效率就会呈现线性下降，把FD_SETSIZE改大的后果就是，大家都慢慢来，什么？都超时了？？！！
3. 内核/用户空间 内存拷贝问题，如何让内核把FD消息通知给用户空间呢？在这个问题上select采取了内存拷贝方法。

### poll
poll其实和select差不多, 不过没有并发数上的限制
### Epoll的提升

把其他模型逐个批判了一下，再来看看Epoll的改进之处吧，其实把select的缺点反过来那就是Epoll的优点了。
1. Epoll没有最大并发连接的限制，上限是最大可以打开文件的数目，这个数字一般远大于2048, 一般来说这个数目和系统内存关系很大，具体数目可以cat /proc/sys/fs/file-max察看。
2. 效率提升，Epoll最大的优点就在于它只管你“活跃”的连接，而跟连接总数无关，因此在实际的网络环境中，Epoll的效率就会远远高于select和poll。
3. 内存拷贝，Epoll在这点上使用了“共享内存”，这个内存拷贝也省略了。

### Epoll为什么高效

Epoll的高效和其数据结构的设计是密不可分的，这个下面就会提到。

首先回忆一下select模型，当有I/O事件到来时，select通知应用程序有事件到了快去处理，而应用程序必须轮询所有的FD集合，测试每个FD是否有事件发生，并处理事件；代码像下面这样：
```c
int res = select(maxfd+1, &readfds, NULL, NULL, 120);
if(res > 0)
{
    for(int i = 0; i < MAX_CONNECTION; i++)
    {
        if(FD_ISSET(allConnection[i],&readfds))
        {
            handleEvent(allConnection[i]);
        }
    }
}
// if(res == 0) handle timeout, res < 0 handle error
```
Epoll不仅会告诉应用程序有I/0事件到来，还会告诉应用程序相关的信息，这些信息是应用程序填充的，因此根据这些信息应用程序就能直接定位到事件，而不必遍历整个FD集合。
```
intres = epoll_wait(epfd, events, 20, 120);
for(int i = 0; i < res;i++)
{
    handleEvent(events[n]);
}
```

#### Epoll关键数据结构

前面提到Epoll速度快和其数据结构密不可分，其关键数据结构就是：
```
structepoll_event {
    __uint32_t events;      // Epoll events
    epoll_data_t data;      // User datavariable
};

typedef union epoll_data {
    void *ptr;
   int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;
```
可见epoll_data是一个union结构体,借助于它应用程序可以保存很多类型的信息:fd、指针等等。有了它，应用程序就可以直接定位目标了。

### 使用Epoll
既然Epoll相比select这么好，那么用起来如何呢？会不会很繁琐啊…先看看下面的三个函数吧，就知道Epoll的易用了。
**intepoll_create(int size);**

生成一个Epoll专用的文件描述符，其实是申请一个内核空间，用来存放你想关注的socket fd上是否发生以及发生了什么事件。size就是你在这个Epoll fd上能关注的最大socket fd数，大小自定，只要内存足够。

**intepoll_ctl(int epfd, intop, int fd, structepoll_event *event);**

控制某个Epoll文件描述符上的事件：注册、修改、删除。其中参数epfd是epoll_create()创建Epoll专用的文件描述符。相对于select模型中的FD_SET和FD_CLR宏。

**intepoll_wait(int epfd,structepoll_event * events,int maxevents,int timeout);**

等待I/O事件的发生；参数说明：

epfd:由epoll_create() 生成的Epoll专用的文件描述符；

epoll_event:用于回传代处理事件的数组；

maxevents:每次能处理的事件数；

timeout:等待I/O事件发生的超时值；

返回发生事件数。

相对于select模型中的select函数。

```c
//   
// a simple echo server using epoll in linux  
//   
// 2009-11-05  
// 2013-03-22:修改了几个问题，1是/n格式问题，2是去掉了原代码不小心加上的ET模式;
// 本来只是简单的示意程序，决定还是加上 recv/send时的buffer偏移
// by sparkling  
//   
#include <sys/socket.h>  
#include <sys/epoll.h>  
#include <netinet/in.h>  
#include <arpa/inet.h>  
#include <fcntl.h>  
#include <unistd.h>  
#include <stdio.h>  
#include <errno.h>  
#include <iostream>  
using namespace std;  
#define MAX_EVENTS 500  
struct myevent_s  
{  
    int fd;  
    void (*call_back)(int fd, int events, void *arg);  
    int events;  
    void *arg;  
    int status; // 1: in epoll wait list, 0 not in  
    char buff[128]; // recv data buffer  
    int len, s_offset;  
    long last_active; // last active time  
};  
// set event  
void EventSet(myevent_s *ev, int fd, void (*call_back)(int, int, void*), void *arg)  
{  
    ev->fd = fd;  
    ev->call_back = call_back;  
    ev->events = 0;  
    ev->arg = arg;  
    ev->status = 0;
    bzero(ev->buff, sizeof(ev->buff));
    ev->s_offset = 0;  
    ev->len = 0;
    ev->last_active = time(NULL);  
}  
// add/mod an event to epoll  
void EventAdd(int epollFd, int events, myevent_s *ev)  
{  
    struct epoll_event epv = {0, {0}};  
    int op;  
    epv.data.ptr = ev;  
    epv.events = ev->events = events;  
    if(ev->status == 1){  
        op = EPOLL_CTL_MOD;  
    }  
    else{  
        op = EPOLL_CTL_ADD;  
        ev->status = 1;  
    }  
    if(epoll_ctl(epollFd, op, ev->fd, &epv) < 0)  
        printf("Event Add failed[fd=%d], evnets[%d]\n", ev->fd, events);  
    else  
        printf("Event Add OK[fd=%d], op=%d, evnets[%0X]\n", ev->fd, op, events);  
}  
// delete an event from epoll  
void EventDel(int epollFd, myevent_s *ev)  
{  
    struct epoll_event epv = {0, {0}};  
    if(ev->status != 1) return;  
    epv.data.ptr = ev;  
    ev->status = 0;
    epoll_ctl(epollFd, EPOLL_CTL_DEL, ev->fd, &epv);  
}  
int g_epollFd;  
myevent_s g_Events[MAX_EVENTS+1]; // g_Events[MAX_EVENTS] is used by listen fd  
void RecvData(int fd, int events, void *arg);  
void SendData(int fd, int events, void *arg);  
// accept new connections from clients  
void AcceptConn(int fd, int events, void *arg)  
{  
    struct sockaddr_in sin;  
    socklen_t len = sizeof(struct sockaddr_in);  
    int nfd, i;  
    // accept  
    if((nfd = accept(fd, (struct sockaddr*)&sin, &len)) == -1)  
    {  
        if(errno != EAGAIN && errno != EINTR)  
        {  
        }
        printf("%s: accept, %d", __func__, errno);  
        return;  
    }  
    do  
    {  
        for(i = 0; i < MAX_EVENTS; i++)  
        {  
            if(g_Events[i].status == 0)  
            {  
                break;  
            }  
        }  
        if(i == MAX_EVENTS)  
        {  
            printf("%s:max connection limit[%d].", __func__, MAX_EVENTS);  
            break;  
        }  
        // set nonblocking
        int iret = 0;
        if((iret = fcntl(nfd, F_SETFL, O_NONBLOCK)) < 0)
        {
            printf("%s: fcntl nonblocking failed:%d", __func__, iret);
            break;
        }
        // add a read event for receive data  
        EventSet(&g_Events[i], nfd, RecvData, &g_Events[i]);  
        EventAdd(g_epollFd, EPOLLIN, &g_Events[i]);  
    }while(0);  
    printf("new conn[%s:%d][time:%d], pos[%d]\n", inet_ntoa(sin.sin_addr), 
            ntohs(sin.sin_port), g_Events[i].last_active, i);  
}  
// receive data  
void RecvData(int fd, int events, void *arg)  
{  
    struct myevent_s *ev = (struct myevent_s*)arg;  
    int len;  
    // receive data
    len = recv(fd, ev->buff+ev->len, sizeof(ev->buff)-1-ev->len, 0);    
    EventDel(g_epollFd, ev);
    if(len > 0)
    {
        ev->len += len;
        ev->buff[len] = '\0';  
        printf("C[%d]:%s\n", fd, ev->buff);  
        // change to send event  
        EventSet(ev, fd, SendData, ev);  
        EventAdd(g_epollFd, EPOLLOUT, ev);  
    }  
    else if(len == 0)  
    {  
        close(ev->fd);  
        printf("[fd=%d] pos[%d], closed gracefully.\n", fd, ev-g_Events);  
    }  
    else  
    {  
        close(ev->fd);  
        printf("recv[fd=%d] error[%d]:%s\n", fd, errno, strerror(errno));  
    }  
}  
// send data  
void SendData(int fd, int events, void *arg)  
{  
    struct myevent_s *ev = (struct myevent_s*)arg;  
    int len;  
    // send data  
    len = send(fd, ev->buff + ev->s_offset, ev->len - ev->s_offset, 0);
    if(len > 0)  
    {
        printf("send[fd=%d], [%d<->%d]%s\n", fd, len, ev->len, ev->buff);
        ev->s_offset += len;
        if(ev->s_offset == ev->len)
        {
            // change to receive event
            EventDel(g_epollFd, ev);  
            EventSet(ev, fd, RecvData, ev);  
            EventAdd(g_epollFd, EPOLLIN, ev);  
        }
    }  
    else  
    {  
        close(ev->fd);  
        EventDel(g_epollFd, ev);  
        printf("send[fd=%d] error[%d]\n", fd, errno);  
    }  
}  
void InitListenSocket(int epollFd, short port)  
{  
    int listenFd = socket(AF_INET, SOCK_STREAM, 0);  
    fcntl(listenFd, F_SETFL, O_NONBLOCK); // set non-blocking  
    printf("server listen fd=%d\n", listenFd);  
    EventSet(&g_Events[MAX_EVENTS], listenFd, AcceptConn, &g_Events[MAX_EVENTS]);  
    // add listen socket  
    EventAdd(epollFd, EPOLLIN, &g_Events[MAX_EVENTS]);  
    // bind & listen  
    sockaddr_in sin;  
    bzero(&sin, sizeof(sin));  
    sin.sin_family = AF_INET;  
    sin.sin_addr.s_addr = INADDR_ANY;  
    sin.sin_port = htons(port);  
    bind(listenFd, (const sockaddr*)&sin, sizeof(sin));  
    listen(listenFd, 5);  
}  
int main(int argc, char **argv)  
{  
    unsigned short port = 12345; // default port  
    if(argc == 2){  
        port = atoi(argv[1]);  
    }  
    // create epoll  
    g_epollFd = epoll_create(MAX_EVENTS);  
    if(g_epollFd <= 0) printf("create epoll failed.%d\n", g_epollFd);  
    // create & bind listen socket, and add to epoll, set non-blocking  
    InitListenSocket(g_epollFd, port);  
    // event loop  
    struct epoll_event events[MAX_EVENTS];  
    printf("server running:port[%d]\n", port);  
    int checkPos = 0;  
    while(1){  
        // a simple timeout check here, every time 100, better to use a mini-heap, and add timer event  
        long now = time(NULL);  
        for(int i = 0; i < 100; i++, checkPos++) // doesn't check listen fd  
        {  
            if(checkPos == MAX_EVENTS) checkPos = 0; // recycle  
            if(g_Events[checkPos].status != 1) continue;  
            long duration = now - g_Events[checkPos].last_active;  
            if(duration >= 60) // 60s timeout  
            {  
                close(g_Events[checkPos].fd);  
                printf("[fd=%d] timeout[%d--%d].\n", g_Events[checkPos].fd, g_Events[checkPos].last_active, now);  
                EventDel(g_epollFd, &g_Events[checkPos]);  
            }  
        }  
        // wait for events to happen  
        int fds = epoll_wait(g_epollFd, events, MAX_EVENTS, 1000);  
        if(fds < 0){  
            printf("epoll_wait error, exit\n");  
            break;  
        }  
        for(int i = 0; i < fds; i++){  
            myevent_s *ev = (struct myevent_s*)events[i].data.ptr;  
            if((events[i].events&EPOLLIN)&&(ev->events&EPOLLIN)) // read event  
            {  
                ev->call_back(ev->fd, events[i].events, ev->arg);  
            }  
            if((events[i].events&EPOLLOUT)&&(ev->events&EPOLLOUT)) // write event  
            {  
                ev->call_back(ev->fd, events[i].events, ev->arg);  
            }  
        }  
    }  
    // free resource  
    return 0;  
}
```