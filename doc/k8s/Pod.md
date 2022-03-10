

[TOC]



## Disruption/PodDisruptionBudgets

https://kubernetes.io/docs/concepts/workloads/pods/disruptions/

PDB 用来保护 Pod ，减少 Voluntary disruptions 对应用的影响。

一个应用的 Pod 通过 label 来筛选

### Voluntary disruptions/involuntary disruptions

Involuntary disruptions 包括：

- 节点所在物理机出现硬件故障
- 集群管理员不小心把 VM 删了
- 云提供商或 hypervisor 故障导致 VM 消失
- 内核错误
- 集群网络出现分区，导致 node 消失
- Node 资源不足导致 pod 被驱逐

除了资源不足的场景，其他场景对用户来说应该很熟悉，它们并不是特定于 k8s 的问题，Involuntary disruption 是不可避免的。

Voluntary disruption 包括由应用程序所有者发起的操作和由集群管理员发起的操作。



PodDisruptionBudgets，这个名字看起来有点奇怪，字面意思是 `Pod 干扰预算`，是什么意思呢？Disruption 我们可以简单地理解就是 Pod 被销毁了，Disruption budgets 意思就是我们允许有多少 Disruption 发生。举个例子，假设我们有一个 Deployment，它下面有 100 个 Pod，我们再给这个 Deployment 的 Pod 创建一个 PodDisruptionBudgets，并把 maxUnavaliable 设置为 10，此时可以理解为我们给这个了这个 Deployment 10 个 Disruption 的预算，每当它的一个 Pod  被销毁时(准确地说应该是被驱逐)，便会消耗 1 个预算，它最多可以消耗 10 个预算，即它最多允许 100 个 Pod 中有 10 个 Pod 处于不可用状态。它的应用场景主要是：在调用 `kubectl drain`对 node 进行维护时，node 会将上面跑的 Pod 给驱逐(evict)掉，驱逐 Pod 本身不是太大的问题，因为被驱逐的 Pod 接下来就会在另外一个 node 运行起来，问题是如果一下子太多的 Pod 被驱逐掉的话，就可能导致服务不稳定，使用 PodDisruptionBudgets，可以让 node 缓慢地驱逐 Pod，使这些被驱逐的 Pod 平滑地切换到其他 node 上。以刚刚的例子来说，当集群管理员使用`kubectl drain`对一个 node 进行维护的时候，即使 Deployment 的 100 个 Pod 都刚好在这个 node 上，也不会发生 100 个 Pod 一下子被驱逐掉导致服务暂时不可用，而是会先驱逐掉 10 个 Pod，等这些 Pod 在其他 node 上运行起来时，才会继续驱逐剩下的 pod。

实际上，我们可以理解为它的作用是为一个应用的 Pod 定义其允许的

