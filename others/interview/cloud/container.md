

## 容器的实现原理

容器的实现主要是用到了操作系统内核的 Cgroups 和 Namespace 机制，使用 Cgroups 来做资源限制，使用 Namespace 来做隔离。

Namespace: 本质上容器就是一个特殊的进程，创建容器的过程就是在调用内核的创建进程的系统调用 clone 时，传递了一些特殊的参数，比如和 Namespace 有关的参数，告诉内核我要创建的这个进程的 PID，Network，Mount 等要有独立的空间，这样创建出来的进程

Cgroups: 操作内核以文件系统的方式暴露出来的系统调用接口