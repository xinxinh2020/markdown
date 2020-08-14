[TOC]

## Static Pod是什么

随Node启动而运行，并且只能在这个Node上运行（不会迁移），并且不会保存到etcd上，而是存放到Node上一个具体的文件中

## Pod中的Pause容器有什么用

1. 用来代表整个Pod，它的状态代表整个Pod的状态，避免了检测各个业务容器的状态的复杂性
2. Pod内多个业务容器可以共享Pause容器的IP地址，挂载的Volume，解决了业务容器之间的网络通信和文件共享问题

## 有几种探针(Probe)，它们的作用是什么

The [kubelet](https://kubernetes.io/docs/admin/kubelet/) uses liveness probes to know when to restart a container. For example, liveness probes could catch a deadlock, where an application is running, but unable to make progress. Restarting a container in such a state can help to make the application more available despite bugs.

The kubelet uses readiness probes to know when a container is ready to start accepting traffic. A Pod is considered ready when all of its containers are ready. One use of this signal is to control which Pods are used as backends for Services. When a Pod is not ready, it is removed from Service load balancers.

The kubelet uses startup probes to know when a container application has started. If such a probe is configured, it disables liveness and readiness checks until it succeeds, making sure those probes don't interfere with the application startup. This can be used to adopt liveness checks on slow starting containers, avoiding them getting killed by the kubelet before they are up and running

## Init Container有什么作用



## Label和Label Selector

 P26

## RC(Replica Set)的作用

P31

## K8s中的Volume和Docker中的Volume有什么区别

- 它是Pod范围的，Pod中所有容器共享
- 它的生命周期和Pod的生命周期相同，和容器的生命周期并无直接关系，当容器终止或重启时，Volume中的数据也不会丢失
- k8s的Volume支持多种类型的Volume，如GlusterFS、Ceph等分布式文件系统

## Deployment的Pod更新策略(spec.strategy)有哪几种

两种：

- Recreate：先杀掉所有正在运行中的Pod，再创建新的Pod，非滚动升级方式
- RollingUpdate：默认方式，滚动进行升级

## DaemonSet的Pod更新策略有哪几种

两种：

- OnDelete: 默认升级策略，更新后新的Pod并不会自动创建，直到用户手动删除旧版本的Pod，才会触发新建操作
- RollingUpdate: 1.6开始，更新后旧版本会被自动杀掉，并自动创建新版本的Pod

## 使用Deployment对Pod进行滚动升级时，Deployment是如何实现Pod的滚动更新的

Deployment是通过ReplicaSet管理Deployment的，当更新Deployment时，系统会创建一个新的ReplicaSet，并将其副本数量扩展到1，然后将旧的ReplicaSet副本数量-1，之后新的ReplicaSet副本数量逐渐增加到预期数量，旧的ReplicaSet副本逐渐减少到0。

在升级过程中，k8s会把Pod副本数控制在一定的范围之内，从1.6开始，Deployment确保可用的副本数不会少于期望（DESIRED）副本数的75%，也即最多有25%的Pod不可用(maxUnavailable)，最多不会比期望副本数多25%，也即最多有25%个浪涌值(maxSurge)

## 更新Deployment的标签选择器(Label Selector)时，会发生什么

一般不鼓励这种做法，可能和其他控制器产生冲突。

添加和修改选择器标签时，必须同步修改Pod的标签。添加后，会创建新的ReplicaSet和Pod，但旧的ReplicaSet和Pod不会被删除，会变成孤立状态

删除选择器标签的时候，ReplicaSet和Pod不会受到任何影响，但被删除的标签仍然会存在于现有的Pod和ReplicaSet上



## RC是如何实现rolling-update的

使用配置文件进行更新时，配置文件需要：

- 使用新的名字
- selector中应用至少有一个标签和旧的RC不同

使用命令直接更新时会修改RC的deployment标签(没有则添加)



## Horizontal Pod AutoScaler(HPA)的机制

HPA控制器可以作用在Deployment,ReplicaSet上，周期性（默认为15s，通过kube-controller-manager的--horizontal-pod-autoscaler-sync-period指定）地监测目标Pod的资源性能指标，通过Metrics Server（Heapster或自定义Metrics Server）的API获取这些数据，目前支持的指标类型：

- Pod资源使用率：CPU使用率等
- Pod自定义指标：如接收的请求数量等
- 外部自定义指标：通常是一个数值，需要容器应用以某种方式提供，如通过http url：/metrics提供

扩缩容算法：`期望副本数=ceil(当前副本数*(当前指标值/期望指标值))`，ceil表示取整

如果定义多种指标，将对每个指标都进行计算，并以最大值为准进行扩缩容操作

计算当前指标值/期望指标值时，异常状态的Pod(如正在删除的Pod)不会计算在内，计算结果接近1时，可以设置一个容忍度（kube-controller-manager的--horizontal-pod-autoscaler-tolerance）让系统不做扩缩容操作。

缺失指标、NotReady状态的Pod会被系统保守地进行平均值计算，系统会假设这些Pod在缩容时消耗了期望值的100%，在扩容时消耗了期望值的0%，从而抑制潜在的扩缩容操作



## Headless Service和普通Service的区别是什么，有什么作用

区别：Service会将客户端请求自动负载均衡到后端的endpoint，而Headless Service则是把后端的enpoint列表返回给客户端，让客户端自己控制负载均衡策略

作用：

- 让客户端(外部访问)获取后端endpoint列表，可以实现自己的负载均衡策略
- 让应用程序(Pod中)知道属于同组服务的其它实例，可以实现分布式集群动态加入新增的节点（Pod）



## Service的NodePort,LoadBalancer,Ingress有什么区别

https://blog.csdn.net/kenkao/article/details/86764788



## Service Account有什么用

它不是给k8s集群的用户用的，它是给Pod里的服务用的



## 网络基础

Pod内容器网络通信：共享网络命名空间

同一个Node内的Pod之间进行通信：通过docker0网桥桥接起来

所有Pod的IP地址都会记录在etcd中，



## Pod的调度策略

- Deployment控制器自动调度
- NodeSelector：调度到带有指定标签的节点
- NodeAffinity：可硬限制（required）也可软限制(preffered)
- PodAffinity：可软限制也可硬限制
- AntiAffinity：Pod互斥性
- Taints和Tolerations：污点的effect包括NoSchedule,PreferNoSchedule（软限制）,NoExecute（不仅不能调度，已调度的Pod也会被驱逐）
- 优先级调度：priorityClassName,用于抢占调度
- DaemonSet
- Job/CronJob，Parallel控制并行度

## Pod跨主机如何通信



## Ingress Http 7层路由

## gRPC相比restful api有什么优势

- 使用protobuff来作为传输的格式
- 数据大小比XML要小3-10倍，序列化和反序列化的速度要快100倍

一个gRPC方法只接受一个Protocol buffer消息类型作为它的请求，并只返回一个Protocol buffer类型作为它的响应



