# Namespace
Namespaces are a feature of the Linux kernel that partitions kernel resources such that one set of processes sees one set of resources while another set of processes sees a different set of resources. The feature works by having the same namespace for a set of resources and processes, but those namespaces refer to distinct resources. Resources may exist in multiple spaces. Examples of such resources are process IDs, hostnames, user IDs, file names, and some names associated with network access, and interprocess communication.

命名空间建立系统的不同视图, 从用户的角度来看, 每一个命名空间应该像一台单独的Linux计算机一样, 有自己的init进程(PID为1), 其他进程的PID依次递增, A和B空间都有PID为1的init进程, 子命名空间的进程映射到父命名空间的进程上, 父命名空间可以知道每一个子命名空间的运行状态, 而子容器与子容器之间是隔离的。从图中我们可以看到，进程3在父命名空间里面PID 为3，但是在子命名空间内，他就是1.也就是说用户从子命名空间 A 内看进程3就像 init 进程一样，以为这个进程是自己的初始化进程，但是从整个 host 来看，他其实只是3号进程虚拟化出来的一个空间而已。

## Namespace类型
1. UTS Namespace
UTS Namespace主要用来隔离nodename和domainname两个系统标识. 在UTS Namespace里面, 每个Namespace允许有自己的hostname.

2. IPC Namespace
用来隔离System V IPC和POSIX message queues. 每一个IPC Namespace都有自己的System V IPC和POSIX message queue.

3. PID Namespace
用来隔离进程ID的. 同样一个进程在不同的PID Namespace里可以拥有不用PID. 这样就可以理解, 在docker container里面, 使用ps -ef经常会发现, 在容器内, 前台运行的进程PID是1, 但是在容器外, 使用ps -ef会发现同样的进程却有不同的PID, 这就是PID Namespace做的事情.

4. Mount Namespace
Mount Namespace用来隔离各个进程看到的挂载点视图. 在不同Namespace的进程中, 看到的文件系统层次是不一样的. 在Mount Namespace中调用mount()和umount()仅仅只会影响当前Namespace内的文件系统, 而对全局的文件系统是没有影响的.

5. User Namespace
User namespace 主要是隔离用户的用户组ID。也就是说，一个进程的User ID 和Group ID 在User namespace 内外可以是不同的。比较常用的是，在宿主机上以一个非root用户运行创建一个User namespace，然后在User namespace里面却映射成root 用户。这样意味着，这个进程在User namespace里面有root权限，但是在User namespace外面却没有root的权限。从Linux kernel 3.8开始，非root进程也可以创建User namespace ,并且此进程在namespace里面可以被映射成 root并且在 namespace内有root权限。

6. Network Namespace
Network namespace 是用来隔离网络设备，IP地址端口等网络栈的namespace。Network namespace 可以让每个容器拥有自己独立的网络设备（虚拟的），而且容器内的应用可以绑定到自己的端口，每个 namesapce 内的端口都不会互相冲突。在宿主机上搭建网桥后，就能很方便的实现容器之间的通信，而且每个容器内的应用都可以使用相同的端口。

# Linux Cgroup
Linux Cgroup(Control Groups)提供了对一组进程及将来子进程的资源限制, 控制和统计的能力, 这些资源包括CPU, 内存, 存储, 网络等. 通过Cgroup, 可以方便地限制某个进程的资源占用, 并且可以实时地监控进程的监控和统计信息.

### Cgroups 中的3个组件
* cgroup 是对进程分组管理的一种机制， 一个cgroup包含一组进程，井可以在这个cgroup上增加 Linux subsystem 的各种参数配置，将一组进程和一组subsystem 的系统参数关联起来。

* subsystem是一组资源控制的模块，一般包含如下几项:
1. blkio设置对块设备(比如硬盘)输入输出的访问控制
2. cpu设置cgroup中进程的CPU被调度的策略
3. cpuacct可以统计cgroup中进程的CPU占用
4. cpuset在多核机器上设置cgroup中进程可以使用的CPU和内存(此处内存仅使用于UMA 架构)
5. devices控制cgroup中进程对设备的访问
6. freezer用于挂起(suspend)和恢复(resume)cgroup中的进程
7. memory用于控制cgroup中进程的内存占用
8. net_els用于将cgroup中进程产生的网络包分类，以便Linux的tc(traffic conoller)可根据分类区分出来自某个cgroup的包并做限流或监控
9. net_prio设置cgroup中进程产生的网络流量的优先级
10. ns这个subsystem比较特殊，它的作用是使cgroup中的进程在新的Namespace中fork新进程(NEWNS)时，创建出一个新的cgroup，这个 cgroup包含新的Namespace中的进程

* hierarchy的功能是把一组cgroup串成一个树状的结构，一个这样的树便是一个hierarchy，通过这种树状结构，Cgroups可以做到继承。比如，系统对一组定时的任务进程通过cgroupl限制了CPU的使用率，然后其中有一个定时dump日志的进程还需要限制磁盘IO，为了避免限制了磁盘IO之后影响到其他进程，就可以创建cgroup2，使其继承于cgroupl井限制磁盘的IO，这样cgroup2便继承了cgroupl中对CPU使用率的限制，并且增加了磁盘IO的限制而不影响到cgroupl中的其他进程。

### 三个组件相互的关系
通过上面的组件的描述我们就不难看出，Cgroups的是靠这三个组件的相互协作实现的，那么这三个组件是什么关系呢？ 

* 系统在创建新的hierarchy之后，系统中所有的进程都会加入到这个hierarchy的根cgroup节点中，这个cgroup根节点是hierarchy默认创建，后面在这个hierarchy中创建cgroup都是这个根cgroup节点的子节点。
* 一个subsystem只能附加到一个hierarchy上面
* 一个hierarchy可以附加多个subsystem
* 一个进程可以作为多个cgroup的成员，但是这些cgroup必须是在不同的hierarchy中
* 一个进程fork出子进程的时候，子进程是和父进程在同一个cgroup中的，也可以根据需要将其移动到其他的cgroup中。