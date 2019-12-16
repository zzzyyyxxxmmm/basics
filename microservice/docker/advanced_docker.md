# 什么是Docker
docker将应用打包成一个镜像, 并且以容器的方式运行. Docker容器将一系列软件包装在一个完整的文件系统中, 这个文件系统包含应用程序运行所需要的一切: 代码, 运行时工具, 系统依赖.

Docker容器具有以下3个特点:
* 轻量级: 在同一台宿主机上的容器共享系统kernel, 这使得它们可以迅速启动而且占用内存极少. 镜像是以分层文件系统构造的, 这可以让他们共享相同的文件, 使得磁盘使用率和镜像下载速度得到提高. 虚拟机会包含一整个操作系统, 容器只包含用户的程序和所有依赖
* 开放: Docker容器基于开放标准, 这使得Docker容器可以运行在主流Linux发行版和Windows操作系统上
* 安全: 容器将各个应用程序隔离开, 这给所有的应用程序提供了一层额外的安全防护

Docker是一个使用了Linux Namespace和Cgroup的虚拟化工具

### 用Go语言实现通过cgroup限制容器的资源
我们知道 Docker是通过 Cgroups实现容器资源限制和监控的，下面以一个实际的容器实 例来看一下 Docker 是如何配置 Cgroups 的。
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

