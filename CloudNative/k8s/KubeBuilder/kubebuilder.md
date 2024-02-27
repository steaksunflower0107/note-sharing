# 初始化 for mac

## prepare

```shell
brew instarll go
brew install docker
brew install kubectl
brew install kustomize
```

## 按照kubebuilder

```shell
brew install kubebuilder
```

## 创建项目

```shell
go mod init 项目名 #没有gopath的情况下需要先使用gomod初始化项目
```

## 脚手架生成框架代码

```shell
kubebuilder init --domain ”项目名“ --skip-go-version-check #kubebuilder init --domain my.domain 
```

这句命令会生成一些文件，包括main.go等文件 --skip-go-version-check 跳过go版本检查（高于1.17的时候）

## 创建API

```shell
kubebuilder create api --group collector.ip --version v1 --kind Collector
```



## 框架目录

```text
nginx-operator
├── config #运行 operator 所需的所有配置文件,目前里面包括 Kustomize YAML 配置文件，后续需要加入 CustomResourceDefinitions(CRD)和一些其他的配置文件
│      ├── default #包含 Kustomize base 文件，用于以标准配置启动 controller
│      ├── rbac #包含运行 controller 所需最小权限的配置文件
│      └── default #包含 Kustomize base 文件，用于以标准配置启动 
├── PROJECT #用于创建新组件的 Kubebuilder 元数据
├── Makefile #用于构建和部署 controller
├── main.go #程序的入口
└── ...
```

# 基本架构

## main.go

 main 文件最开始是 import 一些基本库，尤其是：

- 核心的 [控制器运行时](https://pkg.go.dev/sigs.k8s.io/controller-runtime?tab=doc) 库
- 默认的控制器运行时日志库-- Zap 

```go
package main

import (
    "flag"
    "fmt"
    "os"

    "k8s.io/apimachinery/pkg/runtime"
    _ "k8s.io/client-go/plugin/pkg/client/auth/gcp"
    ctrl "sigs.k8s.io/controller-runtime"
    "sigs.k8s.io/controller-runtime/pkg/cache"
    "sigs.k8s.io/controller-runtime/pkg/log/zap"
    // +kubebuilder:scaffold:imports
)
```

### scheme

每组controller都需要一个scheme，提供kinds与相应的go类型(GVK)之间的映射

```go
var (
    scheme   = runtime.NewScheme()
    setupLog = ctrl.Log.WithName("setup")
)

func init() {

    // +kubebuilder:scaffold:scheme
}
```

![image-20230704095809754](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230704095809754.png)

### manager 

记录所有controller的运行情况，以及设置共享缓存和API服务器的客户端（注意，我们把我们的 Scheme 的信息告诉了 manager）

manager是所有controller和webhooks的运行管制

当要停止一个pod，manager的状态在收到一个graceful shutdown signal时会由running变为停止

```go
func main() {
    var metricsAddr string
    var enableLeaderElection bool
    var probeAddr string
    flag.StringVar(&metricsAddr, "metrics-bind-address", ":8080", "The address the metric endpoint binds to.")
    flag.StringVar(&probeAddr, "health-probe-bind-address", ":8081", "The address the probe endpoint binds to.")
    flag.BoolVar(&enableLeaderElection, "leader-elect", false,
        "Enable leader election for controller manager. "+
            "Enabling this will ensure there is only one active controller manager.")
    opts := zap.Options{
        Development: true,
    }
    opts.BindFlags(flag.CommandLine)
    flag.Parse()

    ctrl.SetLogger(zap.New(zap.UseFlagOptions(&opts)))

    mgr, err := ctrl.NewManager(ctrl.GetConfigOrDie(), ctrl.Options{
        Scheme:                 scheme,
        MetricsBindAddress:     metricsAddr,
        Port:                   9443,
        HealthProbeBindAddress: probeAddr,
        LeaderElection:         enableLeaderElection,
        LeaderElectionID:       "80807133.tutorial.kubebuilder.io",
    })
    if err != nil {
        setupLog.Error(err, "unable to start manager")
        os.Exit(1)
    }
```

## Group & Version & Kinds

### Group & Version

是暴露出来的一系列资源，形式为“Group/Version”,e.g. "policy.k8s.io/v1"

### Kind

每个 API Group-Version 包含一个或多个 API 类型，我们称之为 **Kinds**。

### Resource

 resources（资源） 只是 API 中的一个 Kind 的使用方式。通常情况下，Kind 和 resources 之间有一个一对一的映射。 例如，`pods`  resources对应于 `Pod` kind。但是有时，同一kind可能由多个resources返回。例如，`Scale` Kind 是由所有 `scale` 子资源返回的，如 `deployments/scale` 或 `replicasets/scale`

然而，使用 CRD，每个 Kind 都将对应一个 resources。

注意：resources 总是用小写，按照惯例是 Kind 的小写形式

## Controller

控制器的工作是确保对于任何给定的对象，世界的实际状态（包括集群状态，以及潜在的外部状态，如 Kubelet 的运行容器或云提供商的负载均衡器）与对象中的期望状态相匹配。每个控制器专注于一个根 Kind，但可能会与其他 Kind 交互

这个过程称为 **reconciling**。

在 controller-runtime 中，为特定种类实现 reconciling 的逻辑被称为 [*Reconciler*](https://pkg.go.dev/sigs.k8s.io/controller-runtime/pkg/reconcile)。 Reconciler 接受一个对象的名称，并返回我们是否需要再次尝试（例如在错误或周期性控制器的情况下，如 HorizontalPodAutoscaler）

### reconciler

kubebuilder 搭建了一个基本的 reconciler 结构。几乎每一个调节器都需要记录日志，并且能够获取对象，所以可以直接使用

```go
type CronJobReconciler struct {
    client.Client
    Log    logr.Logger
    Scheme *runtime.Scheme
}
```

`Reconcile` 实际上是对单个对象进行调谐。我们的 Request 只是有一个名字，但我们可以使用 client 从缓存中获取这个对象

我们返回一个空的结果，没有错误，这就向 controller-runtime 表明我们已经成功地对这个对象进行了调谐，在有一些变化之前不需要再尝试调谐



controller-runtime 通过一个名为 [logr](https://github.com/go-logr/logr) 日志库使用结构化的记录日志。正如我们稍后将看到的，日志记录的工作原理是将键值对附加到静态消息中。我们可以在我们的 Reconcile 方法的顶部预先分配一些配对信息，将他们加入这个 Reconcile 的所有日志中。

```go
func (r *CronJobReconciler) Reconcile(req ctrl.Request) (ctrl.Result, error) {
    _ = context.Background()
    _ = r.Log.WithValues("cronjob", req.NamespacedName)

    // your logic here

    return ctrl.Result{}, nil
}
```

最后，我们将 Reconcile 添加到 manager 中，这样当 manager 启动时它就会被启动。

现在，我们只是注意到这个 Reconcile 是在 `CronJob`s 上运行的。以后，我们也会用这个来标记其他的对象。

```go
func (r *CronJobReconciler) SetupWithManager(mgr ctrl.Manager) error {
    return ctrl.NewControllerManagedBy(mgr).
        For(&batchv1.CronJob{}).
        Complete(r)
}
```



4个container:

- 考虑RS的特性，试图反证更新 = 删除+创建，通过kubectl delete 删除
  - 删除一个或两个都是触发了两次事件

- 副本数由3 -> 1时，触发了四次事件

- 副本数由1 -> 3时，触发了四次事件

- 副本数由4 -> 0时，触发了四次事件
- 2副本数
  - strategy 由RollingUpdate -> Recreate ，触发了两次事件，反之依然
- 4副本数
  - 同

2个container:

- kubectl delete删除时，删除一个或两个都是触发了两次事件
- 副本数由3 -> 1时，触发了五次事件 ![image-20230712125619286](/Users/steaksunflower/Library/Application Support/typora-user-images/image-20230712125619286.png)
- 副本数由1 -> 3时，触发了五次事件
- 副本数由3 -> 4时，触发了四次事件
- 副本数由4 -> 0时，触发了五次事件
- 2副本数
  - strategy 由RollingUpdate -> Recreate ，触发了两次事件，反之依然
- 4副本数
  - 同

