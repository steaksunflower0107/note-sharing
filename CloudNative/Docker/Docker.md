# 为什么需要Docker

核心点是解决软件运行的环境配置问题，“环境”包括语言软件包、依赖包、环境变量等。

解决的思路是，在某一软件被安装到某一台机器上时，将其环境也一并安装好

## 传统物理机部署服务

![image-20230527135601539](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230527135601539.png)

服务的可用性欠佳：进程间资源抢占。如果某个进程耗用了100%的CPU资源，其他的进程无法提供服务。或者如果一个进程因为突发异常很多，日志把磁盘打满了，所有的进程都要挂掉。
  既然因为资源共享导致的问题，那么解决方式就是：进程间硬件资源隔离。于是有了虚拟机技术

## 虚拟机

![image-20230527135939685](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230527135939685.png)

虚拟机可以解决环境的配置问题与实现硬件隔离，比如fastDFS无法跑到window系统上，我们可以通过安装一个macOS或Linux系统的来运行它，但这会造成资源和时间上不小的牺牲。

- 虚拟机本身是躺在硬盘里的一个文件，是一个独立的操作系统，每台虚拟机会事先从物理机获取cpu核数，内存， 磁盘，每台虚拟机一般只部署一个应用。从而解决了进程间资源隔离的问题，但该进程无论在虚拟机中执行某个任务用了多少内存，其真实占用的本台机器的内存都需要几百内存才能继续运行
- 虚拟机的启动过程包含一个操作系统启动所需要的一切，如用户登录等
- 启动一个虚拟机，往往需要花费和启动本机同样多的时间甚至更多
- 在解决环境配置这个问题上，如果遇到一些快速迭代的包含大量依赖包的项目，运维的同学需要在每次迭代后，手动配置新的环境与依赖，那么有可能会有遗漏，同时也会耗费较多的人力成本与时间成本

因此，如果把一个虚拟机环境打包到软件中的话，首先软件会变得十分大，用户启动软件的体验感也会下降很多

## Linux容器

基于虚拟机的缺点，在Linux之上研发出一种新的虚拟化技术：Linux容器(Linux Containers)

首先与虚拟机的区别就是它不像一个“独立的操作系统”那么重了，它并不会模拟一个完整的操作系统，而是在底层操作上提供进程级别的隔离和资源管理，容器使用的资源是虚拟的，这种隔离是通过一系列的内核功能来实现的。

- 命名空间(namespaces)

  使得容器内的进程可以拥有自己独立的视图，包括进程ID、网络接口、文件系统、IPC等，由此使得容器内的进程都认为自己处在一个独立的环境中，与其他容器及宿主系统的进程相互隔离，不能直接访问到其他容器的资源

- 控制组

  是一种资源管理机制，用于限制和分配容器中进程可以使用的资源，如CPU、内存、磁盘I/O等，通过控制组，可以对容器内的进程施加资源限制和优先级，以防止容器过度使用系统资源或相互之间产生干扰

这样轻量级的隔离机制使得一个包含环境配置的进程基于虚拟化能够更快的启动、部署和迁移，在资源利用上更加高效

## Docker

![image-20230527140707717](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230527140707717.png)

基于LXD的优点，新的虚拟化技术Docker对LXD进行了封装，提供相关功能的接口，使其能被用户更易使用到其特性，是目前最流行的容器化解决方案

Docker将应用程序和该程序的依赖包打包到一个文件中，当运行该文件时，会生成虚拟容器，包含程序所需的应用环境，虚拟出来一个了物理机，解决了环境配置问题

docker容器技术的核心之一在于**镜像文件**。

镜像文件，通俗的理解就是一个进程运行时依赖的软件文件的集装箱。

应用集群部署时，每台机器首先会拉取指定版本的镜像文件。安装镜像后产生了docker容器。由于所有机器的镜像文件一样，容器的软件版本故而一样。即使开发或运维中途修改了容器的软件版本，但是容器销毁时，软件的改动会随容器的销毁一起湮灭。当应用用已有的镜像文件重新部署时，生成的docker容器跟修改之前的容器完全一样。这也是Infrastructure as code思想带来的好处。

容器如果要升级软件版本，那就修改镜像文件。这样部署时集群内所有的机器重新拉取新的镜像，软件因此跟着一起升级。软件版本混乱的问题，到docker这里，也就得到了完美的解决。

一个疑问：**有了容器技术，生产环境为何还需要部署虚拟机？**

虚拟机能做到硬件资源的彻底隔离，docker不行。虚拟机 和 docker各取长处，最佳CP。

# Docker的应用

- 测试、体验等一次性环境

  可以用docker去测试具备特殊条件、甚至一定风险的软件

- 为云服务提供弹性

  docker通过微服务中的监控机制反馈的实时信息，可以实现的docker数量的动态扩缩容，由此在保证服务可用性的基础上降低了耗电成本

- 在微服务架构中作为服务实例

  通过创建多个docker，一台机器上就可以部署多个服务实例，减少了机器成本

  

# Docker架构

![image-20230527142202058](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230527142202058.png)

- 镜像(Image)：相当于是一个root文件系统，比如官方镜像ubuntu:16.04就包含了一套ubuntu16.04最小系统的root文件系统
- 容器(container)：**镜像与实例的关系，类似于面向对象程序设计中类的实例的关系**，镜像是静态的定义，容器是镜像运行时的实体，故容器可以被创建、启动、停止、删除、暂停等
- 仓库(Repository)：Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。
  Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用，用于保存不同镜像的不同版本
- 客户端(Client)：Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的[守护进程](https://www.coonote.com/cplusplus-note/linux-daemons.html)通信。
- 主机(Host)：一个物理或者虚拟的机器用于执行Docker守护进程和容器
- Registry：一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。
- Machine：Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。

## 卷的概念

卷（Volume）是一种用于持久化存储数据的机制，它允许容器在其生命周期中将数据存储在主机的特定位置，而不是仅限于容器内部的文件系统。卷提供了一种方便的方式来共享数据和持久化容器的状态。

- 数据持久化：Docker 卷允许容器中的数据在容器销毁后仍然保持持久化。这对于存储数据库文件、日志、配置文件等需要持久化的数据非常有用。

- 共享数据：多个容器可以共享同一个卷，这使得容器之间可以方便地共享文件和数据。这对于微服务架构或多个容器之间的协作非常有用。

- 数据备份和迁移：通过使用卷，可以轻松备份和迁移容器中的数据。卷可以与容器解耦，使得备份和迁移过程更加简单。

- 主机绑定：卷可以与主机文件系统中的特定目录或文件进行绑定。这样，容器中的数据将直接写入到主机上的相应位置，从而实现主机和容器之间的数据共享。

- 独立于容器的生命周期：卷的生命周期可以超过容器的生命周期。当容器被删除时，卷仍然存在，可以供其他容器使用。

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。

Docker 容器通过 Docker 镜像来创建。



# Docker底层技术

## Linux上的命名空间(namespaces)

这是实现Docker进程隔离的核心技术，通过Linux中命名空间的强大特性，使得每个容器都有自己的单独的命令空间，类似于让每个容器跑在自己的操作系统中，由此各容器之间互不影响

- pid命名空间

  不同的用户进程是通过pid命名空间隔离开的，且不同的名字空间中可以有相同的pid，所有的LXC进程在Docker中的父进程为Docker进程，每个LXC的进程具有不同的命名空间，同时由于允许嵌套，因此可以很方便的实现嵌套的Docker容器

- net命名空间

  有了pid命名空间，每个命名空间中的pid实现了相互隔离，但还需要解决跑在容器里的进程不能共享主机的网络端口的问题，网络隔离是由net命名空间实现的，每个net命名空间有独立的网络设备、IP地址、路由表，/proc/net目录。由此使得每个容器的网络得以隔离开来

  ```apl
  /proc/net 目录是 Linux 操作系统中的一个特殊目录，用于提供有关网络状态和网络配置的信息。它是一个虚拟文件系统，不包含实际的文件，而是通过内核动态地生成和更新文件内容。
  
  在 /proc/net 目录下，你可以找到多个文件，每个文件对应一个特定的网络相关信息。以下是一些常见的文件和其对应的功能：
  
  /proc/net/dev： 提供有关网络接口的统计信息，如接收和发送的数据包数量、错误计数等。
  
  /proc/net/if_inet6： 显示 IPv6 地址的分配情况，包括接口名称、IPv6 地址、前缀长度等。
  
  /proc/net/arp： 显示 ARP（Address Resolution Protocol）缓存表，包含 IP 地址和对应的 MAC 地址。
  
  /proc/net/route： 显示 IP 路由表的信息，包括目标网络、网关、接口、跃点数等。
  
  /proc/net/tcp 和 /proc/net/tcp6： 显示 TCP 连接的信息，如本地地址和端口、远程地址和端口、状态等。
  
  /proc/net/udp 和 /proc/net/udp6： 显示 UDP 连接的信息，如本地地址和端口、远程地址和端口等。
  
  /proc/net/icmp 和 /proc/net/icmp6： 显示 ICMP（Internet Control Message Protocol）的统计信息，如 ICMP 报文类型、发送和接收的数量等。
  
  通过读取和解析 /proc/net 目录下的文件，可以获取关于网络接口、路由、连接和协议的信息，对于网络调试、性能监控和故障排除非常有用。
  ```

  

  Docker中默认使用veth的方式，将容器中的虚拟网卡同host上的一个Docker网桥docker0连接在一起

  ```apl
  在 Docker 中，默认使用的网络类型是 veth（Virtual Ethernet）。veth 设备是一对虚拟的网络设备，用于连接 Docker 容器和宿主机的网络命名空间。
  
  当创建一个 Docker 容器时，Docker 引擎会在宿主机上创建一对 veth 设备，其中一个端口连接到宿主机的网络命名空间，另一个端口则被分配给容器的网络命名空间。这样，宿主机和容器之间就可以通过 veth 设备进行网络通信。
  
  Docker 为每个容器分配一个唯一的网络命名空间，并为容器中的每个网络接口分配一个 veth 设备。这样，每个容器都可以拥有自己独立的网络栈，包括独立的 IP 地址、路由表和网络配置。
  
  当容器需要与宿主机或其他容器通信时，Docker 使用 Linux 桥接（bridge）技术来连接不同的 veth 设备。Linux 桥接是一种虚拟网络设备，用于将多个网络设备连接在一起，形成一个共享网络的桥接网络。
  
  通过 veth 设备和 Linux 桥接技术，Docker 实现了容器之间的网络隔离和通信，并提供了一种灵活的网络架构，使得容器可以与宿主机和其他容器之间进行安全、高效的网络交互。
  ```

  

- ipc命令空间

  容器中进程交互还是采用了 Linux 常见的进程间交互方法(interprocess communication – IPC), 包括信号量、消息队列和共享内存等。然而同 VM 不同的是，容器的进程间交互实际上还是 host 上具有相同 pid 命名空间中的进程间交互，因此需要在 IPC 资源申请时加入名字空间信息，每个 IPC 资源有一个唯一的 32位 id。

- mnt命名空间

  类似 chroot，将一个进程放到一个特定的目录执行。mnt 命名空间允许不同命名空间的进程看到的文件结构不同，这样每个命名空间中的进程所看到的文件目录就被隔离开了。同 chroot 不同，每个名字空间中的容器在 /proc/mounts 的信息只包含所在名字空间的 mount point

- uts命名空间

  UTS(“UNIX Time-sharing System”) 命名空间允许每个容器拥有独立的 hostname 和 domain name, 使其在网络上可以被视作一个独立的节点而非主机上的一个进程。

- user命名空间

  每个容器可以有不同的用户和组 id, 也就是说可以在容器内用容器内部的用户执行程序而非主机上的用户

通过隔离命名空间，Docker能够确保每个容器运行在一个相对独立的环境中，提供了隔离性和安全性。



## 控制组

控制组(cgroup)是Linux内核的一个特性，主要是将共享的资源隔离、限制、审计等，由此实现分配到容器的资源是可控的，来避免同一宿主机的多个容器之间发生系统资源的竞争

Linux内核由2.6.24开始支持这一特性

可控的资源有

- 内存
- CPU
- 审计管理

## 命名空间与控制组总结

我们可以说命名空间侧重于是实现了进程级别的隔离性，使得各容器使用的资源互不干扰（不是绝对的），而控制组侧重于实现资源管理，包括分配、调度等



## 联合文件系统

联合文件系统是一种分层、轻量级、高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件之下

联合文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

不同 Docker 容器就可以共享一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。

### Docker中的应用：AUFS

AUFS支持为每一个成员目录(类似于Git的分支)，可以设定为：

- 只读(readonly)
- 读写(readwrite)
- 写出(whiteout-able)

AUFS中有一个类似分层的概念，对只读的权限的分支可以逻辑上的进程增量地修改(不影响只读部分)

Docker目前支持的联合文件系统种类包括AUFS、btrfs、vfs和DeviceMapper

## 容器格式

最初，Docker 采用了 LXC 中的容器格式。自 1.20 版本开始，Docker 也开始支持新的 libcontainer 格式，并作为默认选项。



# Dockerfile

一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令

- FROM：格式为<仓库>/<镜像>:<标签(版本)>，若不完整则默认使用官方镜像

  - 使用其他 Docker Hub 用户或组织的镜像：

  ```
  FROM username/repo:tag
  ```

  - 使用第三方镜像仓库的镜像：

  ```
  FROM registry.example.com/repository/image:tag
  ```

  - 使用本地构建的基础镜像：

  ```
  FROM my-base-image:tag
  ```

- WORKDIR

  - 指定接下来的shell语句的执行目录，若没有会自行创建

- MAINTAINER

  ```MAINTAINER docker_user docker_user@email.com```

- COPY

  - 将宿主机的文件拷贝到coker中 

    ```COPY src/ /app       # 将宿主的src文件拷贝到app文件中```

- ADD

  - 类似于COPY，但可以是URL
  - 仅在需要自动解压缩的场合

- RUN 
  - 运行shell语句
  - 在构建镜像时运行，不会在容器执行时运行
  - 常用于安装软件包、配置环境等操作
  - 每个 `RUN` 指令都会在当前镜像的一个新的中间层上执行命令，并将结果保存下来
- CMD
  - 也是运行shell语句
  - 容器启动后运行的命令
  - 可以以JSON数组的方式指定，也可以以命令行方式指定
  - 只能在Dockerfile中出现一次
- ENTRYPOINT
  - 类似于CMD
- ![image-20230601185708776](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230601185708776.png)

- EXPOSE
  - 使构建的镜像服务端暴露某端口，使容器在运行时监听，功互联系统使用，启动容器时需要docker run -P
- VOLUME
  - VOLUME用于创建挂载点，即向基于所构建镜像创始的容器添加卷

- USER
  - 用于指定运行镜像所使用的用户
  - 使用USER指定用户后，Dockerfile 中其后的命令 RUN、CMD、ENTRYPOINT 都将使用该用户。镜像构建完成后，通过 docker run 运行容器时，可以通过 -u 参数来覆盖所指定的用户。

# Docker Compose

负责快速在集群中部署分布式应用。Compose 则允许用户在一个模板（YAML 格式）中定义一组相关联的应用容器（被称为一个 project，即项目），例如一个 Web 服务容器再加上后端的数据库服务容器等。

![image-20230601191855574](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230601191855574.png)

# Docker 命令

-d：使得容器在后台运行，此时想要进入容器，可以通过以下指令进入：

- docker attach
- **docker exec**：推荐使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。



# Docker使用

## Docker容器互联

- 创建一个新的Docker网络

  ```bash
  docker network create -d bridge demo
  ```

**-d**：参数指定 Docker 网络类型，有 bridge、overlay。

- 桥接网络（Bridge Network）：这是 Docker 默认创建的网络类型。每个容器都连接到一个桥接网络，可以通过容器名称或 IP 地址进行通信。桥接网络可以在同一主机上的容器之间提供基本的网络连接。
- Overlay 网络（Overlay Network）：Overlay 网络允许在多个 Docker 主机之间创建跨主机的网络，用于容器之间的通信。这种网络类型通常用于构建容器集群和跨主机的应用程序部署。

![image-20230601204947892](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230601204947892.png)

运行两个容器到该docker网络中：

```bash
docker run -itd --name demo3 --network demo ubuntu /bin/bash
```

...

安装ping：

```bash
docker exec -it demo3 /bin/bash
apt-get update
apt install iputils-ping
```





# 常见问题

```
denied: requested access to the resource is denied
```

解决：docker tag 镜像名 账户名/镜像名



