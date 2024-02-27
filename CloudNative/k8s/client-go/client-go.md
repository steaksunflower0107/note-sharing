# 目录结构

```javascript
client-go/
  ├── discovery
  ├── dynamic
  ├── informers
  ├── kubernetes
  ├── listers
  ├── plugin
  ├── rest
  ├── scale
  ├── tools
  ├── transport
  └── util
```

- `discovery`：提供 `DiscoveryClient` 发现客户端
- `dynamic`：提供 `DynamicClient` 动态客户端
- `informers`：每种 kubernetes 资源的 Informer 实现
- `kubernetes`：提供 `ClientSet` 客户端
- `listers`：为每一个 Kubernetes 资源提供 Lister 功能，该功能对 Get 和 List 请求提供只读的缓存数据
- `plugin`：提供 OpenStack、GCP 和 Azure 等云服务商授权插件
- `rest`：提供 `RESTClient` 客户端，对 Kubernetes API Server 执行 RESTful 操作
- `scale`：提供 `ScaleClient` 客户端，用于扩容或缩容 Deployment、ReplicaSet、Relication Controller 等资源对象
- `tools`：提供常用工具，例如 SharedInformer、Reflector、DealtFIFO 及 Indexers。提供 Client 查询和缓存机制，以减少向 kube-apiserver 发起的请求数等
- `transport`：提供安全的 TCP 连接，支持 Http Stream，某些操作需要在客户端和容器之间传输二进制流，例如 exec、attach 等操作。该功能由内部的 spdy 包提供支持
- `util`：提供常用方法，例如 WorkQueue 功能队列、Certificate 证书管理等



# 包

## k8s.io/api/core/v1

1. **定义核心资源对象结构**：该包包含了 Kubernetes 中的核心资源对象（如 Pod、Service、Namespace、ConfigMap、Secret 等）的结构定义。这些结构定义了资源对象的属性、元数据和规范，用于描述和配置 Kubernetes 中的各种资源。
2. **操作核心资源对象**：该包提供了与核心资源对象的交互的方法和函数。你可以使用这些方法来创建、读取、更新和删除核心资源对象，以及执行其他与核心资源对象相关的操作。
3. **与核心资源对象的编组和解组**：该包提供了用于将核心资源对象编组为 JSON 或其他格式的方法，以及将已编组的对象解组回资源对象的方法。这使得你可以在与 Kubernetes API 进行通信时对资源对象进行序列化和反序列化操作。



# 架构

![image-20230709231445450](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230709231445450.png)

## 控制器逻辑

![image-20230709231559448](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230709231559448.png)

# RESTClient

- RESTClient ：最基础的客户端，提供最基本的封装
- Clientset：是一个Client的集合，包含了所有k8s内置的资源类型，由此可以去通过其提供的api去操作Pod、Deployment、Service这些资源
- dynamicClient：是动态的客户端，可以操作任意类型的k8s资源，包括CRD定义的资源
- DiscoveryClient：用于发现K8S提供的资源组、资源版本和资源信息，比如kubectl api-resource

