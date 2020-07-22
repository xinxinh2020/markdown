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

