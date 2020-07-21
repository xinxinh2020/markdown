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



有以下值：

- Pending：Pod已经被k8s系统接受，但Pod还有容器没有被创建。包括了被调度前的时间和下载容器镜像的时间
- Running：Pod已经被调度到Node上，所有的容器都已经被创建，至少有一个容器还在运行中（或者在启动或重启中）
- Succeeded：Pod中的所有容器都以成功状态停止，并且不会再次重启
- Failed：Pod中的所有容器都已经停止，并且至少有一个容器是以失败停止的（以非0状态退出或被系统强制停止）
- Unknown：由于某种原因无法获得Pod的状态，一般是和Pod所在的Host出现通信问题导致

kubelet可以对运行中的容器执行3种探测：

- livenessProbe：kubelet使用这个探针来知道什么时候来重启一个容器
- readinessProbe
- startupProbe



livenessProbe：想要在探测失败的时候把容器Kill掉重新启动的时候

readinessProbe：设置了readinessProbe探测后，只有探测成功时Pod才会被加入到Service的可用列表中，开始对外提供服务。如果Pod的容器在启动时需要加载大量的数据或配置，最好就给它设置一个readinessProbe，因为加载的时间可能很长，在这段时间内应该暂时不对外提供服务（实际上一般也提供不了服务）。

如果只是想让容器成功启动前不接收服务请求，那么readinessProbe可以设置成和livenessProbe一样，如果想要容器能够暂时停止对外服务进行维护的话，readinessProbe应该设置成和livenessProbe不一样探针。

如果只是想要在Pod被删除后让请求不要再发送到这个Pod上，是可以不用设置readinessProbe的，因为在删除时，Pod会自动把它置为不可用，所以在Pod容器删除过程中，这个Pod是不会收到请求的。

readinessProbe：v1.16后支持。