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




Pod的phase有以下值：

- Pending：Pod已经被k8s系统接受，但Pod还有容器没有被创建。包括了被调度前的时间和下载容器镜像的时间
- Running：Pod已经被调度到Node上，所有的容器都已经被创建，至少有一个容器还在运行中（或者在启动或重启中）
- Succeeded：Pod中的所有容器都以成功状态停止，并且不会再次重启
- Failed：Pod中的所有容器都已经停止，并且至少有一个容器是以失败停止的（以非0状态退出或被系统强制停止）
- Unknown：由于某种原因无法获得Pod的状态，一般是和Pod所在的Host出现通信问题导致

容器的状态：

- Running：表示容器正在正常运行中，postStart钩子会在容器进入Running状态前执行

- Waiting：容器默认的状态，只要不是Running和Terminated状态就是这个状态，拉取镜像的时候也是这个状态

- Terminated：表示容器已经执行完并停止运行。执行完有两种情况，一种是正常退出，另外一种是因为某种原因失败退出，通过reason和exit code查看容器停止的原因和状态码。在一个容器进入Terminated状态前，preStop钩子会被执行

  ```shell
     State:          Terminated
       Reason:       Completed
       Exit Code:    0
       Started:      Wed, 30 Jan 2019 11:45:26 +0530
       Finished:     Wed, 30 Jan 2019 11:45:26 +0530
  ```

  



kubelet可以对运行中的容器执行3种探测：

- livenessProbe：kubelet使用这个探针来知道什么时候来重启一个容器
- readinessProbe
- startupProbe



livenessProbe：想要在探测失败的时候把容器Kill掉重新启动的时候

readinessProbe：设置了readinessProbe探测后，只有探测成功时Pod才会被加入到Service的可用列表中，开始对外提供服务。如果Pod的容器在启动时需要加载大量的数据或配置，此时既不想把容器kill掉，又不想接收请求，最好就给它设置一个readinessProbe，因为加载的时间可能很长，在这段时间内应该暂时不对外提供服务（实际上一般也提供不了服务）。

如果只是想让容器成功启动前不接收服务请求，那么readinessProbe可以设置成和livenessProbe一样，如果想要容器能够暂时停止对外服务进行维护的话，readinessProbe应该设置成和livenessProbe不一样探针。

如果只是想要在Pod被删除后让请求不要再发送到这个Pod上，是可以不用设置readinessProbe的，因为在删除时，Pod会自动把它置为不可用，所以在Pod容器删除过程中，这个Pod是不会收到请求的。

readinessProbe：v1.16后支持。



探针支持的属性：

- initialDelaySeconds：默认值是0，最小值是0，容器启动后等待多久，liveness or readiness开始第1次执行
- periodSeconds：多久执行一次探针，默认值是10，最小值是1
- timeoutSeconds：探针执行超时时间，默认值是1，最小值是1
- successThreshold：失败后执行多少次探针成功才算成功，默认值是1，最小值是1，对于liveness来说必须是1
- failureThreshold：探针连续失败多少次后才算失败，默认是3，最小值是1



Http探针额外的属性：

- host：主机名，默认值是Pod的IP
- sheme：http还是https，默认是http
- path：url地址
- httpHeaders：头信息
- port：端口，范围1-65535



### Restart policy

restartPolicy 取值：

- Always：默认取值
- OnFailure：
- Never：

kubernetes restart pod时只会重启在同一个node上，重启的延迟为：10s，20s，40s...上限为5分钟。在成功执行10分钟后重置该延迟。一旦一个Pod被调度到一个Node上，它就和这个node绑定了，不会被调度到其他node上。

当出现磁盘故障或Node故障时，Pod的阶段会变为Failed，如果它受controller管理的话，controller会在其他Node重新创建它(而不是在本机尝试重启)

## Init Container

一个Pod可以有一个或多个Init Container，它会在业务容器启动前运行。

Init容器有以下特点：

- 必须要能运行结束
- 一个Init容器成功结束后，另一个Init容器才能开始运行

Init容器失败也会触发Pod的重启策略（成功不会）。

Init容器的作用：

- init容器可以延长业务容器的启动，知道业务容器启动所需的前置配置完成后再让业务容器跑起来
- init容器可以运行一些可能不太安全的工具或代码，将这些工具和代码从业务容器中分离出来有利于提高业务容器的安全

Pod重启后，Init容器需要重新执行

可以修改Init容器的镜像，但会触发Pod重启，业务容器的镜像改变只会触发该业务容器重启