[TOC]

# 硬件设置

## 将BIOS调至高性能模式

需要注意的是，高性能模式并不适合所有应用，因为它更耗电。然而，在进行基准测试和性能敏感性集群中，建议使用高性能模式。

## PCIe选择

使用适合网络适配器的PCIe，大多数情况下需要使用：

- PCIe第3代
- 速度8GT/s
- 宽度为x8或x16

需要注意的是如果有x16，可以用在x8网络适配器上。

## NUMA相关

对于相关的应用，要使用Mellanox网卡使用的PCIe总线直接连接的CPU核。可以通过如下命令查看PCIe总线使用的NUMA Node：

```shell
cat /sys/devices/pci0000\:00/0000\:00\:02.0/numa_node
0
```

通过以下命令将进程绑定到指定的CPU核上运行：

```shell
taskset -cp 0-13,28-41 45118 # 45118为进程号 0-13,28-41为node 0关联的CPU核
```



## 禁用tx-nocache-copy

tx-nocache-copy是一个特性，它绕过本地缓存直接把用户态的数据写入到内存中，新的内核已经默认禁用了该特性，当有些Linux发行版默认是打开的。查看该特性是否启用可以用以下命令：

```shell
[root@power27 test-classes]# ethtool --show-offload ib0 | grep tx-nocache-copy
tx-nocache-copy: off
```





参考资料：https://community.mellanox.com/s/article/performance-tuning-for-mellanox-adapters