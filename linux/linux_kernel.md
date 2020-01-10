# Tasks performed by the kernel
* **Process scheduling**: A computer has one or more central processing units (CPUs), which execute the instructions of programs. Like other UNIX systems, Linux is a preemptive multitasking operating system, Multitasking means that multiple processes (i.e., running programs) can simultaneously reside in mem- ory and each may receive use of the CPU(s). Preemptive means that the rules governing which processes receive use of the CPU and for how long are deter- mined by the kernel process scheduler (rather than by the processes them- selves).
* **Memory management**: While computer memories are enormous by the stan- dards of a decade or two ago, the size of software has also correspondingly grown, so that physical memory (RAM) remains a limited resource that the ker- nel must share among processes in an equitable and efficient fashion. Like most modern operating systems, Linux employs virtual memory management (Section 6.4), a technique that confers two main advantages:

– Processes are isolated from one another and from the kernel, so that one process can’t read or modify the memory of another process or the kernel.

– Only part of a process needs to be kept in memory, thereby lowering the memory requirements of each process and allowing more processes to be held in RAM simultaneously. This leads to better CPU utilization, since it increases the likelihood that, at any moment in time, there is at least one process that the CPU(s) can execute.
* Provision of a file system: The kernel provides a file system on disk, allowing files to be created, retrieved, updated, deleted, and so on.
* Creation and termination of processes: The kernel can load a new program into memory, providing it with the resources (e.g., CPU, memory, and access to files) that it needs in order to run. Such an instance of a running program is termed a process. Once a process has completed execution, the kernel ensures that the resources it uses are freed for subsequent reuse by later programs.
* Access to devices: The devices (mice, monitors, keyboards, disk and tape drives, and so on) attached to a computer allow communication of information between the computer and the outside world, permitting input, output, or both. The kernel provides programs with an interface that standardizes and simplifies access to devices, while at the same time arbitrating access by multiple processes to each device.
* Networking: The kernel transmits and receives network messages (packets) on behalf of user processes. This task includes routing of network packets to the target system.
* Provision of a system call application programming interface (API): Processes can request the kernel to perform various tasks using kernel entry points known as system calls. The Linux system call API is the primary topic of this book. Section 3.1 details the steps that occur when a process performs a system call.

# Kernel mode and user mode
Modern processor architectures typically allow the CPU to operate in at least two different modes: user mode and kernel mode (sometimes also referred to as supervisor mode). Hardware instructions allow switching from one mode to the other. Corre- spondingly, areas of virtual memory can be marked as being part of user space or kernel space. When running in user mode, the CPU can access only memory that is marked as being in user space; attempts to access memory in kernel space result in a hardware exception. When running in kernel mode, the CPU can access both user and kernel memory space.

Certain operations can be performed only while the processor is operating in kernel mode. Examples include executing the halt instruction to stop the system, accessing the memory-management hardware, and initiating device I/O opera- tions. By taking advantage of this hardware design to place the operating system in kernel space, operating system implementers can ensure that user processes are not able to access the instructions and data structures of the kernel, or to perform operations that would adversely affect the operation of the system.

# Directories and links
A directory is a special file whose contents take the form of a table of filenames coupled with references to the corresponding files. This filename-plus-reference association is called a link, and files may have multiple links, and thus multiple names, in the same or in different directories.
Directories may contain links both to files and to other directories. The links between directories establish the directory hierarchy shown in Figure 2-1.
Every directory contains at least two entries: . (dot), which is a link to the direc- tory itself, and .. (dot-dot), which is a link to its parent directory, the directory above it in the hierarchy. Every directory, except the root directory, has a parent. For the root directory, the dot-dot entry is a link to the root directory itself (thus, /.. equates to /).

## Symbolic links
Like a normal link, a symbolic link provides an alternative name for a file. But whereas a normal link is a filename-plus-pointer entry in a directory list, a symbolic link is a specially marked file containing the name of another file. (In other words, a symbolic link has a filename-plus-pointer entry in a directory, and the file referred to by the pointer contains a string that names another file.) This latter file is often called the target of the symbolic link, and it is common to say that the symbolic link “points” or “refers” to the target file. When a pathname is specified in a system call, in most circumstances, the kernel automatically dereferences (or synonymously, follows) each symbolic link in the pathname, replacing it with the filename to which it points. This process may happen recursively if the target of a symbolic link is itself a symbolic link. (The kernel imposes limits on the number of dereferences to handle the possibility of circular chains of symbolic links.) If a symbolic link refers to a file that doesn’t exist, it is said to be a dangling link.
Often hard link and soft link are used as alternative terms for normal and sym- bolic links. The reasons for having two different types of links are explained in Chapter 18.

参考: 
1. [link1](https://www.cnblogs.com/itech/archive/2009/04/10/1433052.html)
2. [link2](https://www.ibm.com/developerworks/cn/linux/l-cn-hardandsymb-links/index.html)

A process inherits its current working directory from its parent process. A login shell has its initial current working directory set to the location named in the home directory field of the user’s password file entry. The shell’s current working directory can be changed with the cd command.

# Interprocess Communication and Synchronization
* signals, which are used to indicate that an event has occurred;
* pipes (familiar to shell users as the | operator) and FIFOs, which can be used to tansfer data between processes;
* sockets, which can be used to transfer data from one process to another, either on the same host computer or on different hosts connected by a network;
* file locking, which allows a process to lock regions of a file in order to prevent other processes from reading or updating the file contents;
* message queues, which are used to exchange messages (packets of data) between processes;
* semaphores, which are used to synchronize the actions of processes; and
* shared memory, which allows two or more processes to share a piece of memory. When one process changes the contents of the shared memory, all of the other processes can immediately see the changes.

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

### Kernel接口
l 首先， 要创建并挂载一个 hierarchy Ccgroup树)，如下。
```mkdir cgroup-test #创建一个 hierarchy 挂载点```
```sudo mount -t cgroup -o none,name=cgroup-test cgroup-test ./cgroup-test #挂载 一个 hierarchy```
```ls ./cgroup test #挂载后我们就可以看到系统在这个目录下生成了一些默认文件```

```cgroup. clone children cgroup. procs cgroup. sane_behavior notify_on_release release agent tasks```
这些文件就是这个hierarchy中cgroup 根节点的配置项，上面这些文件的含义分别如下。
* cgroup.clone_children, cpuset 的 subsystem 会读取这个配置文件，如果这个值是 I C默 认是 0)，子 cgroup 才会继承父 cgroup 的 cpuset 的配置 。
* cgroup.procs 是树中 当前节点cgroup中的进程组ID，现在的位置是在根节点，这个文件中会有现在系统中所有进程组的ID。
* notify_on_release和release_agent会一起使用。 notify_on_release标识当这个cgroup最 后一个进程退出的时候是否执行了release_agent; release_agent则是一个路径，通常用作进程退出之后自动清理掉不再使用的cgroup。
* tasks标识该cgroup下面的进程ID，如果把一个进程ID写到tasks文件中，便会将相应的进程加入到这个cgroup中。

# /proc
Linux下的/proc文件系统是由内核提供的，它其实不是一个真正的文件系统, 只包含了系统运行时的信息(比如系统内存、mount设备信息、一些硬件配直等)，它只存在于内存中，而不占用外存空间。它以文件系统的形式，为访问内核数据的操作提供接口。实际上，很多系统工具都是简单地去读取这个文件系统的某个文件内容，比如lsmod，其实就是cat /proc/modules。

```
wjk32111@ubuntu:~$ ls /proc/
1       13    174   187    19659  212   2246  2628  33    565  72         diskstats    loadavg       swaps
10      1324  175   188    197    213   225   2638  334   567  793        dma          locks         sys
1002    1327  176   189    198    2131  226   27    34    57   798        driver       mdstat        sysrq-trigger
1012    1330  177   19     199    2135  227   2717  341   58   8          execdomains  meminfo       sysvipc
1015    1361  178   190    2      214   228   2729  35    59   85         fb           misc          thread-self
1018    1383  1789  1901   20     215   229   2762  36    60   86         filesystems  modules       timer_list
102423  1384  179   191    200    216   23    28    3838  61   863        fs           mounts        timer_stats
102855  15    1790  1914   201    217   230   285   384   62   86519      interrupts   mpt           tty
104543  150   1796  1915   202    218   231   2853  3862  63   9          iomem        mtrr          uptime
107384  151   1797  1919   203    2182  232   286   391   64   acpi       ioports      net           version
107881  1515  18    192    204    219   24    29    394   647  asound     irq          pagetypeinfo  version_signature
11      152   180   193    205    22    25    2924  489   648  buddyinfo  kallsyms     partitions    vmallocinfo
110185  153   1800  194    206    220   2544  3     5     65   bus        kcore        sched_debug   vmstat
110216  154   181   195    207    2206  2577  3030  52    651  cgroups    keys         schedstat     zoneinfo
110225  155   182   1953   208    221   26    3056  54    656  cmdline    key-users    scsi
110805  156   183   196    209    222   260   3074  55    66   consoles   kmsg         self
110806  16    184   19601  21     223   2612  3085  56    661  cpuinfo    kpagecgroup  slabinfo
110816  17    185   19622  210    2237  262   3135  563   683  crypto     kpagecount   softirqs
12      173   186   19639  211    224   2624  316   564   7    devices    kpageflags   stat
```
| /proc/N         | PID为N的进程信息                     |
|-----------------|--------------------------------------|
| /proc/N/cmdline | 进程启动命令                         |
| /proc/N/cwd     | 链接到进程当前工作目录               |
| /proc/N/environ | 进程环境变量列表                     |
| /proc/N/exe     | 链接到进程的执行命令文件             |
| /proc/N/fd      | 包含进程相关的所有文件描述符         |
| /proc/N/maps    | 与进程相关的内存映射信息             |
| /proc/N/mem     | 指代进程持有的内存，不可读           |
| /proc/N/root    | 链接到进程的根目录                   |
| /proc/N/stat    | 进程的状态                           |
| /proc/N/statm   | 进程使用的内存状态                   |
| /proc/N/status  | 进程状态信息，比stat/statm更具可读性 |
| /proc/self/     | 链接到当前正在运行的进程             |

# AUFS
```
mkdir aufs && \
cd aufs && \
mkdir container-layer && \
mkdir image-layer1 && \
mkdir image-layer2 && \
mkdir image-layer3 && \
mkdir image-layer4 && \
mkdir mnt

echo "I am container layer" > container-layer/container-layer.txt
echo "I am image layer 1" > image-layer1/image-layer1.txt
echo "I am image layer 2" > image-layer2/image-layer2.txt
echo "I am image layer 3" > image-layer3/image-layer3.txt
echo "I am image layer 4" > image-layer4/image-layer4.txt

sudo mount -t aufs -o dirs=./container-layer:./image-layer4:./image-layer3:./image-layer2:./image-layer1 none ./mnt


jikangwang@ubuntu:~/aufs$ tree
.
├── container-layer
│   └── container-layer.txt
├── image-layer1
│   └── image-layer1.txt
├── image-layer2
│   └── image-layer2.txt
├── image-layer3
│   └── image-layer3.txt
├── image-layer4
│   └── image-layer4.txt
└── mnt
    ├── container-layer.txt
    ├── image-layer1.txt
    ├── image-layer2.txt
    ├── image-layer3.txt
    └── image-layer4.txt

6 directories, 10 filesa

查看aufs系统读写权限
jikangwang@ubuntu:/sys/fs/aufs$ cat si_7f5c402df8f6a9ed/*
/home/jikangwang/aufs/container-layer=rw
/home/jikangwang/aufs/image-layer4=ro
/home/jikangwang/aufs/image-layer3=ro
/home/jikangwang/aufs/image-layer2=ro
/home/jikangwang/aufs/image-layer1=ro
64
65
66
67
68
/home/jikangwang/aufs/container-layer/.aufs.xino

echo -e "\nwrite to mnt's image-layer1.txt" >> ./mnt/image-layer4.txt

jikangwang@ubuntu:~/aufs$ cat mnt/image-layer4.txt 
I am image layer 4

write to mnt's image-layer1.txt

此处, mnt只是一个虚拟挂载点, 因此, 接下来还需要继续去寻找文件修改到底在什么位置

jikangwang@ubuntu:~/aufs$ cat image-layer4/image-layer4.txt 
I am image layer 4

原文件内容没有变化

jikangwang@ubuntu:~/aufs/container-layer$ ls
container-layer.txt  image-layer4.txt

jikangwang@ubuntu:~/aufs/container-layer$ cat image-layer4.txt 
I am image layer 4

write to mnt's image-layer1.txt

也就是说, 当尝试向mnt/image-layer4.txt文件进行写操作的时候, 系统首先在mnt目录下查找名为image-layer4.txt的文件, 将其拷贝到read-write层的container-layer目录中, 接着对container-layer目录中的image-layer4.txt文件进行写操作. 
```
