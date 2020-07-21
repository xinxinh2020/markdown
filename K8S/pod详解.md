[TOC]

一个Pod上的容器共享存储、网络以及运行时配置信息。

Pod中运行一组紧耦合的容器。

Pod是k8s中的原子调度单元。

Service通过*LabelSelector* 来匹配Pod

Service自带负载均衡功能，会把请求分发到后端的Pod中，在Pod更新时，它只会把请求分发到那些可用的Pod上。

如果没有Node的可用内存可以满足Pod请求的内存，则Pod会调度失败，一直Pending

## 资源限制

### 内存限制

https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource/

如果没有为容器指定内存限制：

- 容器没有内存使用上限，当它把Node的可用内存用完时，会触发OOM Killer，在OOM Kill时，这个容器进程很可能被杀掉
- 如果容器所在的namespace有一个默认的内存限制，那么容器会自动继承该限制

容量限制是加在container上的，pod没有容量限制的设置，pod的容量限制等于其所有container的限制之和



### CPU限制

https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/

如果没有为容器指定CPU限制：

- 容器可以将Node上的CPU占满
- 如果容器所在的namespace有一个默认的内存限制，那么容器会自动继承该限制



配置limits和requests成不同值有以下好处：

- 在请求突发时可以有一定的资源可以使用
- 可以把用于突发场景的资源限制在一定的量上



## QoS

3种QoS：

- Guaranteed：cpu/mem 的limits和requests一样
- Burstable：不满足Guaranteed的要求，并且至少有一个容器指定了requests
- BestEffort：不指定cpm/mem的limits和requests

## 生命周期