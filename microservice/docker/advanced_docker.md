# 什么是Docker
docker将应用打包成一个镜像, 并且以容器的方式运行. Docker容器将一系列软件包装在一个完整的文件系统中, 这个文件系统包含应用程序运行所需要的一切: 代码, 运行时工具, 系统依赖.

Docker容器具有以下3个特点:
* 轻量级: 在同一台宿主机上的容器共享系统kernel, 这使得它们可以迅速启动而且占用内存极少. 镜像是以分层文件系统构造的, 这可以让他们共享相同的文件, 使得磁盘使用率和镜像下载速度得到提高. 虚拟机会包含一整个操作系统, 容器只包含用户的程序和所有依赖
* 开放: Docker容器基于开放标准, 这使得Docker容器可以运行在主流Linux发行版和Windows操作系统上
* 安全: 容器将各个应用程序隔离开, 这给所有的应用程序提供了一层额外的安全防护

Docker是一个使用了Linux Namespace和Cgroup的虚拟化工具

### 通过cgroup来限制stress进程的资源
1. ```mkdir cgroup-test```
2. ```sudo mount -t cgroup -o none,name=cgroup-test cgroup-test ./cgroup-test```
3. ```ls ./cgroup-test```

```
jikangwang@ubuntu:~/cgroup-test$ ls
cgroup.clone_children  #cpuset的subsystem会读取这个配置文件, 如果这个值是1(默认是0), 子cgroup才会继承父cgroup的cpuset的配置

cgroup.sane_behavior   
cgroup.procs     #cgroup.procs是树中当前节点cgroup的groupid, 现在的位置是在根节点, 这个文件中会有现在系统中所有进程组的ID     
notify_on_release #notify_on_realease标识当这个cgroup最后一个进程退出的时候是否执行了release_agent; release_agent则是一个路径, 通常用作进程退出之后自动清理掉不再使用的cgroup
release_agent     
tasks #标识该cgroup下面的进程ID, 如果把一个进程ID写到tasks文件中, 便会将相应的进程加入到这个cgroup中
```

4. 在上面创建hierarchy的时候, 这个hierarchy并没有关联到任何的subsystem, 所以没办法通过那个hierarchy中的cgroup节点限制进程的资源占用, 其实系统默认已经为每个subsystem创建了一个默认的hierarchy, 比如memory的hierarchy.

```
mount | grep memory
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
```
5. ```cd /sys/fs/cgroup/memory```
6. ```stress --vm-bytes 200m --vm-keep -m 1```
7. ```sudo mkdir test-limit-memory && cd test-limit-memory```
8. ```sudo sh -c "echo "100m" > memory.limit_in_bytes"```
9. ```sudo sh -c "echo $$ > tasks"```
10. ```stress --vm-bytes 200m --vm-keep -m 1```

### 用Go语言实现通过cgroup限制容器的资源
我们知道 Docker是通过 Cgroups实现容器资源限制和监控的，下面以一个实际的容器实例来看一下Docker是如何配置 Cgroups 的。
```
docker run -m 设置内存限制

sudo docker run -itd m 128m ubuntu
957459145e9092618837cf94alcb356e206f2f0da560b40cb31035e442d3dfll

docker 会为每个容器在系统的 hierarchy 中创建 cgroup

cd /sys/fs/cgroup/memory/docker/957459145e9092618837cf94alcb356e206f2f0da560b40cb310 35e442d3dfl1

cat memory.limit_in_bytes
134217728

cat memory.usage_in_bytes
430080
```
可以看到, Docker通过为每个容器创建cgroup, 并通过cgroup去配置资源限制和资源监控。

在Namespace容器的基础上, 加上cgroup的限制, 实现一个demo, 使其能够具有限制容器内存的功能
```go
package main

import (
	"fmt"
	"io/ioutil"
	"os"
	"os/exec"
	"path"
	"strconv"
	"syscall"
)

// 挂载了memory subsystem的hierarchy的根目录位置
const cgroupMemoryHierarchyMount = "/sys/fs/cgroup/memory"

func main() {
	/*
	sudo ls -lt /proc/self/exe
	lrwxrwxrwx 1 root root 0 Dec 26 16:06 /proc/self/exe -> /bin/ls
	正在运行的进程的软连接
	*/
	if os.Args[0] == "/proc/self/exe" {
		// 容器进程
		fmt.Printf("current pid %d\n", syscall.Getpid())
		cmd := exec.Command("sh", "-c", `stress --vm-bytes 200m --vm-keep -m 1`)
		cmd.SysProcAttr = &syscall.SysProcAttr{}
		cmd.Stdin = os.Stdin
		cmd.Stdout = os.Stdout
		cmd.Stderr = os.Stderr
		if err := cmd.Run(); err != nil {
			fmt.Println(err)
			os.Exit(1)
		}
	}

	cmd := exec.Command("/proc/self/exe")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		fmt.Println("Error", err)
		os.Exit(1)
	} else {
		// 得到fork出来进程映射在外部命名空间的pid
		fmt.Printf("%v\n", cmd.Process.Pid)
		// 在系统默认创建挂载了 memory subsystem 的hierarchy上创建cgroup
		os.Mkdir(path.Join(cgroupMemoryHierarchyMount, "testmemorylimit"), 0755)
		// 将容器进程加入到这个cgroup中
		ioutil.WriteFile(path.Join(cgroupMemoryHierarchyMount, "testmemorylimit", "tasks"), []byte(strconv.Itoa(cmd.Process.Pid)), 0644)
		// 限制cgroup进程使用
		ioutil.WriteFile(path.Join(cgroupMemoryHierarchyMount, "testmemorylimit", "memory.limit_in_bytes"), []byte("100m"), 0644)
		cmd.Process.Wait()
	}
}
```

# UFS
每一个Docker image都是由一系列read-only layer组成的。image layer的内容都存储在Docker hosts filesystem的/var/lib/docker/aufs/diff目录下。而/var/lib/docker/aufs/layers 目录, 则存储着image layer如何堆找这些layer的metadata。

启动一个container的时候, Docker会为其创建一个read-only的init layer, 用来存储与这个容器内环境相关的内容; Docker还会为其创建一个read-write的layer来执行所有的写操作.

container layer的mount目录也是/var/lib/docker/aufs/mnt. container的metadata和配置文件都存放在/var/lib/docker/containers/container-id目录中. container的read-write layer存储在/var/lib/docker/aufs/diff/目录下. 即使容器停止, 这个可读写层依然存在, 因而重启容器不会丢失数据, 只有当一个容器被删除的时候, 这个可读写层才会一起删除.

# 构造容器

## 构造实现run命令版本的容器
```
git clone https://github.com/xianlubird/mydocker.git 
git checkout code-3.1
```

./mydocker run -ti /bin/sh

cmd.Start()会根据参数fork出一个shell进程, 这个时候/proc/self/exe就是指向这个进程(mydocker), 这时候再执行自己+init初始化, 挂载proc系统, 方便通过ps查看当前进程资源情况