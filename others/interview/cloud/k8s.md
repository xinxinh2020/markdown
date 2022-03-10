[TOC]

## Static Pod是什么

随Node启动而运行，并且只能在这个Node上运行（不会迁移），并且不会保存到etcd上，而是存放到Node上一个具体的文件中

## Pod Qos 设计和实现

https://zhuanlan.zhihu.com/p/110979231

kubernetes 中有三种 Qos，分别为：

- `Guaranteed`：pod 的 requests 与 limits 设定的值相等；
- `Burstable`：pod requests 小于 limits 的值且不为 0；
- `BestEffort`：pod 的 requests 与 limits 均为 0；

三者优先级（依次递增）：BestEffort -> Burstable -> Guaranteed

- 在调度时调度器只会根据 request 值进行调度
- 当系统 OOM上时对于处理不同 OOMScore 的进程表现不同，OOMScore 是针对 memory 的，当宿主上 memory 不足时系统会优先 kill 掉 OOMScore 值高的进程，可以使用 `$ cat /proc/$PID/oom_score` 查看进程的 OOMScore。OOMScore 的取值范围为 [-1000, 1000]，`Guaranteed` pod 的默认值为 -998，`Burstable` pod 的值为 2~999，`BestEffort` pod 的值为 1000，也就是说当系统 OOM 时，首先会 kill 掉 `BestEffort` pod 的进程，若系统依然处于 OOM 状态，然后才会 kill 掉 `Burstable` pod，最后是 `Guaranteed` pod；
- cgroup 的配置不同，kubelet 为会三种 Qos 分别创建对应的 QoS level cgroups，`Guaranteed` Pod Qos 的 cgroup level 会直接创建在 `RootCgroup/kubepods` 下，`Burstable` Pod Qos 的创建在 `RootCgroup/kubepods/burstable` 下，`BestEffort` Pod Qos 的创建在`RootCgroup/kubepods/BestEffort` 下

如果不指定 request，会被分配一个很小的数值，cpu 为 2m(cpu.shares=2)

如果资源充足，可将 QoS pods 类型均设置为`Guaranteed`。用计算资源换业务性能和稳定性，减少排查问题时间和成本。如果想更好的提高资源利用率，业务服务可以设置为Guaranteed，而其他服务根据重要程度可分别设置为Burstable或Best-Effort。

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

## PodDisruptionBudget 的作用是什么

https://segmentfault.com/a/1190000021856623

## 网络基础

Pod内容器网络通信：共享网络命名空间

同一个Node内的Pod之间进行通信：通过docker0网桥桥接起来

所有Pod的IP地址都会记录在etcd中，



## 默认调度器的工作原理

调度器的作用就是为 Pod 寻找一个最适合的 Node。具体的调度流程分为 Predicate 和 Priority，如下图所示：

<img src="https://static001.geekbang.org/resource/image/bb/53/bb95a7d4962c95d703f7c69caf53ca53.jpg" alt="img" style="zoom: 4%;" />

Informer 会 watch 与调度相关的对象，当一个 Pod 对象被创建出来以后，调度器就会通过 Pod Informer 的 Handler，将这个带调度的 Pod 添加进调度队列。

Predicate: 进行过滤得到一组 Node。有以下几种策略

- GeneralPredicates: 宿主机的 CPU 和 内存等是否够用，只计算 request 的资源；Pod 申请的宿主机端口是否已经被使用；nodeSelector/nodeAffinity 是否匹配
- 与 Volume 相关的过滤规则：挂载的 PV 是否已经被挂载到其它 Node; PV 数量是否已经超过限制；Pod 申请的 PV 的 nodeAffinity 是否跟节点匹配。
- 与宿主机相关的过滤规则：污点；内存是否不够充足
- Pod 相关的过滤规则：PodAffinity/AntiAf

Priority: 为上一步得到的 Pod 打分，从 0-10分，取得分最高者。

- 优先选择空闲资源多的
- 优先选择调度后资源差（如 CPU 和 内存使用率的差距）小的
- 优先选择 NodeAffinity/PodAffinity 匹配规则多的
- 优先选择镜像已经拉好的

## Pod 调度失败后怎么办

正常情况下，当一个 Pod 调度失败后，它就会被暂时“搁置”起来，直到 Pod 被更新，或者集群状态发生变化，调度器才会对这个 Pod 进行重新调度。但当一个高优先级的 Pod 调度失败后，该 Pod 并不会被“搁置”，而是会“挤走”某个 Node 上的一些低优先级的 Pod 。这样就可以保证这个高优先级 Pod 的调度成功，这个便是 Pod 的优先级和抢占机制，需要给 Pod 设置一个 PriorityClass, 值越大优先级越高，最大值为 10 亿。

优先级高的 Pod 会在调度时优先处理，并且在调度失败时会触发抢占

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

## 一个 PVC 可以给一个 Deployment 的两个 Pod 挂载吗

如果用户删除被某 Pod 使用的 PVC 对象，该 PVC 申领不会被立即移除。 PVC 对象的移除会被推迟，直至其不再被任何 Pod 使用。 此外，如果管理员删除已绑定到某 PVC 申领的 PV 卷，该 PV 卷也不会被立即移除。 PV 对象的移除也要推迟到该 PV 不再绑定到 PVC



## PersistentVolume 对象的回收策略

- Retain: 当 PersistentVolumeClaim 对象 被删除时，PersistentVolume 卷仍然存在，对应的数据卷被视为"已释放（released）"。 由于卷上仍然存在这前一申领人的数据，该卷还不能用于其他申领。 管理员可以通过下面的步骤来手动回收该卷：
  1. 删除 PersistentVolume 对象。与之相关的、位于外部基础设施中的存储资产 （例如 AWS EBS、GCE PD、Azure Disk 或 Cinder 卷）在 PV 删除之后仍然存在。
  2. 根据情况，手动清除所关联的存储资产上的数据。
  3. 手动删除所关联的存储资产；如果你希望重用该存储资产，可以基于存储资产的 定义创建新的 PersistentVolume 卷对象。
- Delete: 删除动作会将 PersistentVolume 对象从 Kubernetes 中移除，同时也会从外部基础设施（如 AWS EBS、GCE PD、Azure Disk 或 Cinder 卷）中移除所关联的存储资产
- Recycle: 回收策略 `Recycle` 已被废弃。取而代之的建议方案是使用动态供应

## PersistentVolume 的 Phase

- Available -- a free resource that is not yet bound to a claim
- Bound -- the volume is bound to a claim
- Released -- the claim has been deleted, but the resource is not yet reclaimed by the cluster
- Failed -- the volume has failed its automatic reclamation

## k8s 的 update 和 patch 一个资源对象的区别

## Pod 的状态（Phase）
有多个容器时，这个时候有一个容器是 ready，另一个还在 waiting，此时 Pod 的 Phase 是什么(Running)

## Headless Service 是什么

Headless Service 不需要分配一个 VIP，而是可以直接以 DNS 记录的方式解析出被代理 Pod 的 IP 地址。

当你按照这样的方式创建了一个 Headless Service 之后，它所代理的所有 Pod 的 IP 地址，都会被绑定一个这样格式的 DNS 记录：`<pod-name>.<svc-name>.<namespace>.svc.cluster.local`

## StatefulSet 是如何实现拓扑状态和存储状态管理的

首先，StatefulSet 的控制器直接管理的是 Pod。这是因为，StatefulSet 里的不同 Pod 实例，不再像 ReplicaSet 中那样都是完全一样的，而是有了细微区别的。比如，每个 Pod 的 hostname、名字等都是不同的、携带了编号的。而 StatefulSet 区分这些实例的方式，就是通过在 Pod 的名字里加上事先约定好的编号。

其次，Kubernetes 通过 Headless Service，为这些有编号的 Pod，在 DNS 服务器中生成带有同样编号的 DNS 记录。只要 StatefulSet 能够保证这些 Pod 名字里的编号不变，那么 Service 里类似于 web-0.nginx.default.svc.cluster.local 这样的 DNS 记录也就不会变，而这条记录解析出来的 Pod 的 IP 地址，则会随着后端 Pod 的删除和再创建而自动更新。这当然是 Service 机制本身的能力，不需要 StatefulSet 操心。

最后，StatefulSet 还为每一个 Pod 分配并创建一个同样编号的 PVC。这样，Kubernetes 就可以通过 Persistent Volume 机制为这个 PVC 绑定上对应的 PV，从而保证了每一个 Pod 都拥有一个独立的 Volume。

## 各种控制器是如何实现版本管理的

deployment 通过 replicaset，其它控制器通过 controllerrevision

