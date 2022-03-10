[TOC]



云原生时代，K8s 可以看作为云原生的操作系统，容器可以看着进程，而容器镜像则是程序安装包，Pod 的就是进程组

容器设计模式：当用户想在一个容器里跑多个功能不相关的应用时，应该优先考虑它们是不是更应该被描述成一个 Pod 里的多个容器

Pod 扮演的是传统部署环境里“虚拟机”的角色，可以把 Pod 看成传统环境里的“机器”、把容器看作是运行在这个“机器”里的“用户程序”，凡是调度、网络、存储，以及安全相关的属性，基本上是 Pod 级别的。

Pod 的恢复过程（restartPolicy），永远都是发生在当前节点上，而不会跑到别的节点上去

Headless Service 不需要分配一个 VIP，而是可以直接以 DNS 记录的方式解析出被代理 Pod 的 IP 地址。

Job 的 restartPolicy=OnFailure 时，离线作业失败后，Job Controller 就不会去尝试创建新的 Pod。但是，它会不断地尝试重启 Pod 里的容器；restartPolicy=Never 时，离线作业失败后 Job Controller 就会不断地尝试创建一个新 Pod

一个 RoleBinding 可以绑定一个 Role 到多个 Subject（如 ServiceAccount） 上, role 定义一组资源的操作权限，ServiceAccount 只有 Name 和 Namespace 两个属性

Operator 本质上就是一个控制器

PersistentVolumeController：不断地查看当前每一个 PVC，是不是已经处于 Bound（已绑定）状态。如果不是，那它就会遍历所有的、可用的 PV，并尝试将其与这个“单身”的 PVC 进行绑定。这样，Kubernetes 就可以保证用户提交的每一个 PVC，只要有合适的 PV 出现，它就能够很快进入绑定状态，从而结束“单身”之旅

当 PV 和 PVC 绑定起来时，就会开始执行 Volume 的 “两阶段处理” 流程 ： Attach 和 Mount 阶段

- Attach: 将块存储（磁盘）挂到宿主机上。由 Volume Controller 中的 AttachDetachController 控制，该控制器位于 Master  节点的 kube-controller-manager, Attach 操作只需要调用公有云或者具体存储项目的 API，并不需要在具体的宿主机上执行操作
- Mount: 将磁盘设备格式化并挂载到一个宿主机目录, 发生在 Pod 对应的宿主机

FlexVolume : 将编写好的存储插件（可执行的二进制或脚本文件）放在所有宿主机的指定目录。Attach 和 Mount 阶段会去所在宿主机通过命令行的方式调用存储插件。（如下图）PV 的 driver 字段用来定位存储插件的存放位置，option 字段会作为参数传给存储插件

```shell
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-flex-nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  flexVolume:
    driver: "k8s/nfs"
    fsType: "nfs"
    options:
      server: "10.10.0.25" # 改成你自己的NFS服务器地址
      share: "export"
```

