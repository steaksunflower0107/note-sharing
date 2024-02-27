# k8s架构

![image-20230602104223800](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230602104223800.png)

- 核心层：对外提供API用于构建部署上层的应用，对内提供插件式应用执行环境

- 应用层：部署(无状态应用、有状态应用、批处理任务、集群应用)

  - 无状态应用：指web的逻辑服务、前端资源等，即请求打过来后不需要保持任何状态，k8s中通常使用 Deployment、ReplicaSet 或 StatefulSet 资源对象
  - 有状态应用：数据库、缓存、消息队列，k8s中通常使用 StatefulSet 资源对象。

  - 批处理任务：
    - 批处理任务一次性处理一组任务或作业，这些任务通常是相同的或相似的，并且可以在相对较长的时间内完成。例如，数据批处理任务可以涉及对大量数据的转换、处理或分析。
    - 通常对任务没有实时性的响应要求，利用闲置的计算资源

- 管理层：系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态 Provision 等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy 等）

  - 系统度量：对于集群中的基础设施进行监控和度量的功能，获取相应设施的健康状况、资源利用率、性能指标等，如CPU、内存和磁盘的使用率、网络流量等
  - 自动化：主要是实现容器自动扩缩容的弹性，减少手动操作和管理的工作量
    - 动态provision：是k8s自带的功能，允许在需要时自动创建和管理存储卷
  - 策略管理：
    - RBAC（Role-Based Access Control）：RBAC 是 Kubernetes 中的一种访问控制机制，用于定义和管理用户、组和服务账号的权限。RBAC 允许管理员根据角色和角色绑定来定义细粒度的权限策略，从而限制用户和服务账号在集群中的访问和操作权限。
    - Quota（资源配额）：Quota 可用于限制命名空间（Namespace）中的资源使用量。通过设置资源配额，可以控制 CPU、内存、存储等资源的使用限制，以确保集群中的各个命名空间不会无限制地消耗资源。
    - PSP（Pod Security Policies）：PSP 是用于控制和约束 Pod 安全配置的策略。通过定义 PSP，管理员可以限制 Pod 使用的特权、挂载的存储卷、容器镜像来源等，以增强集群的安全性。
    - NetworkPolicy（网络策略）：NetworkPolicy 允许管理员定义网络访问策略，用于控制 Pod 之间和 Pod 与集群内外其他资源之间的网络通信。通过 NetworkPolicy，可以限制流量的源、目标 IP、端口等属性，实现网络隔离和安全性的增强。

- 接口层：kubectrl命令行工具，客户端SDK以及集群联邦

- 生态系统：在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴

  - Kubernetes 外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS 应用、ChatOps 等
  - Kubernetes 内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等



## API对象

API对象是用于描述和管理集群状态的基本构建块。API对象是Kubernetes API的核心概念，用于定义和表示集群中的各种资源和操作。

每个API对象都有一个唯一的标识符，由以下几个字段组成：

1. `apiVersion`：指定API对象的版本，它定义了该对象的字段和行为。例如，`apiVersion: v1`表示使用的是Kubernetes核心API的版本1。
2. `kind`：指定API对象的类型或种类。例如，`kind: Pod`表示该API对象是一个Pod对象。这里划分了高层与底层API对象
3. `metadata`：包含与API对象相关的元数据，如名称、命名空间、标签等。
4. `spec`：指定API对象的规范或配置。它描述了对象的**期望状态或所需行为**。每种API对象的`spec`字段都有特定的属性。
5. `status`：包含API对象的当前状态信息，由Kubernetes系统维护和更新。每种API对象的`status`字段都有特定的属性。

常见API对象：

1. Pod：最小的可调度和可部署的单元，用于运行容器。
2. Deployment：用于定义应用程序的部署策略和管理应用程序的副本数量。
3. Service：用于暴露应用程序的网络服务，提供负载均衡和服务发现功能。
4. ReplicaSet：确保指定数量的Pod副本正在运行，并在需要时进行扩缩容。
5. StatefulSet：用于管理有状态应用程序的部署，确保每个Pod都有唯一的标识和稳定的网络标识。
6. DaemonSet：确保每个节点上都运行一个Pod副本，常用于部署守护进程或运行特定的系统任务。
7. Job：用于运行一次性任务或批处理作业，确保任务成功完成。
8. Namespace：用于在集群中创建虚拟的隔离环境，帮助组织和管理资源。
9. ConfigMap：用于存储配置数据，以便在Pod中使用。
10. Secret：用于存储敏感信息，如密码、密钥等。



# POD

不同的团队各自开发构建自己的容器镜像，在部署的时候组合成一个微服务对外提供服务。

Pod是在k8s集群中运行和部署应用或服务的最小单元，支持多容器

同一个Pod中，共享:

- 网络地址(共享相同的网络空间实现)
- 文件系统(共享数据卷实现)

## 控制器

业务分类：

长期伺服型(long-running) --> Deployment

批处理型(batch) --> JOB

节点后台支撑型(node-daemon) --> DaemonSet

有状态应用型(stateful application) --> StatefulSet

## 副本控制器(Replication Controller)

是用于基于监控保证运行中的pod数是指定的副本数量，若达不到，则自动开新的pod满足指定数，若超出，则杀死多余pod，保证至少有一个pod在运行，提升高可用性

- 需要通过RC运行pod
- 适用于长期私服型

## 副本集(Replice Set)

是新一代的RC，能支持更多种类的匹配模式。副本集对象一般不单独使用，而是作为 Deployment 的理想状态参数使用。

## 部署(Deployment)

部署是对k8s集群的一次操作，如创建一个新的服务、更新一个服务，滚动升级一个服务

在实际上，是创建了一个新的RS并逐渐将副本数加到指定数量，而将旧的RS减少到0的复合操作

### Deployment 与Replice Set的关系

Deployment 定义了应用程序的期望状态，而 ReplicaSet 负责实际创建和管理符合该期望状态的 Pod 副本。Deployment 提供了更高级、更灵活的控制和管理能力，而 ReplicaSet 则是实现这些功能的底层组件。

两者类似于类和对象的关系

- Deployment提供了Pod副本的模板，并提供相关的控制和管理副本的功能(类中的方法)
- ReplicaSet是根据Deployment定义创建的

### Replice Set和Pod的关系

类似于管家与住户的关系

- ReplicaSet 是一个控制器对象，类似于一个容器，用于管理一组 Pod 副本的创建和维护。它定义了一组条件或规则，以确保有指定数量的 Pod 副本在运行中。
- Pod 则是 ReplicaSet 中的元素或成员。Pod 是 Kubernetes 中最小的调度单位，包含一个或多个容器，它们共享网络和存储空间，并作为一个逻辑单元来运行应用程序。

## 服务(Service)

RC、RS 和 Deployment 只是保证了支撑服务的微服务 Pod 的数量，但是没有解决如何访问这些服务的问题

一个Pod往往是运行一个服务的多个实例，但随时可能在一个节点上停止，在另一节点以一个新的IP启动一个新的Pod，因此不能确定新Pod的IP和端口

而这需要类似于**负载均衡、服务发现**对客户端发来的请求所做的工作，进行服务绑定使得请求访问到后端对应的服务实例，而这**在k8s中是用Service对象来实现**的

每个 Service 会对应一个集群内部有效的虚拟 IP，集群内部会通过虚拟 IP 访问一个服务，虚拟IP的实现方法有：

- 集群内部的虚拟 IP：在 Kubernetes 集群内部，每个节点都会分配一个虚拟 IP 地址，用于对外提供服务。这些虚拟 IP 地址通常由负载均衡器（如 kube-proxy）管理，并通过 IPVS 或 iptables 等技术将请求流量转发到正确的后端 Pod。

- 外部负载均衡器的虚拟 IP：在一些云平台或基础设施提供商中，可以使用外部负载均衡器来为 Kubernetes 服务分配一个虚拟 IP 地址。这个虚拟 IP 由负载均衡器管理，并将流量转发到集群中的节点或 Pod。

而负载均衡，在k8s中是使用的是由Kube-proxy实现的，它是一个分布式代理服务器，在k8s中每个节点都有这个



#### 域名

当您创建一个 Service 对象时，Kubernetes 会自动在集群的 DNS 服务中为该 Service 分配一个域名。这个域名的格式通常是 `<service-name>.<namespace>.svc.cluster.local`，其中 `<service-name>` 是您创建的 Service 的名称，`<namespace>` 是 Service 所在的命名空间

##### pod访问Service

1.访问Service的域名

```bash
curl http://my-service.namespace.svc.cluster.local
```

2.访问cluster IP

获取 Service 的 Cluster IP 地址：

```bash
kubectl get service my-service -n namespace -o jsonpath='{.spec.clusterIP}'
```

给Cluster IP 发送请求：

```bash
curl http://<cluster-ip>
```

3.Nodeport类型下，访问NodeIP + NodePod

```bash
curl http://<Node-IP>:<NodePort>
```





一般来说svc可以分为四类：`Headless`、`ClusterIP`、`NodePort`、`LoadBalancer`

![image-20230614100349094](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230614100349094.png)

#### Headless Service

- 在配置文件中将cluster ip置为None，即为Headless Service类型
- 不会使用kube-proxy，使用的是DNS域名解析做负载均衡
- 该服务域名解析的结果就是与其所关联的pod的IP

#### ClusterIP Service

每一个node都有一个kube-proxy，它为service实现了一个虚拟IP （clusterIP）



## 任务(Job)

是k8s用于控制批处理型任务的API对象，Job 管理的 Pod 根据用户的设置把任务成功完成就自动退出了。成功完成的标志根据不同的 spec.completions 策略而不同：

- 单Pod型任务有一个Pod成功就标志完成
- 定数成功型任务保证有N个任务全部成功
- 工作队列型任务根据应用确认的全局成功而标志成功

## 后台支撑服务集(DaemonSet)

不同于长期伺服型和批处理型服务的核心是在业务应用，后台支撑型服务的核心是在k8s中的节点(物理机或虚拟机)，要保证每个节点上都有一个此类Pod运行，

