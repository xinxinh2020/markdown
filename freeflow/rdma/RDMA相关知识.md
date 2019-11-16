



# 相关知识

- DMA指一个设备可以在没有CPU干预的情况下访问主机内存的能力了，RDMA是指在没有CPU干预的情况下访问远程主机内存的能力

- 双边操作SEND/RECEIVE多用于连接控制报文，单边操作READ/WRITE多用于数据报文

- 支持RDMA的网络协议：IB、RoCE和iWARP

  - RoCE：交换机使用标准的以太网交换机，网卡使用支持RDMA的网卡。它的包的外层头是以太网头，而内层头是IB头

- 三种网络协议都可以通过verbs API来使用。在linux系统中，它是libibverbs库和verbs内核模块

- Linux内核本身支持RDMA，内核自带的OFED和MLNX-OFED的区别：

  ```shell
  The difference is that the MLNX-OFED contain features/commits that doesn't exist in the upstream kernel;
  * Changes that weren't submitted (yet)
  * Changes that can't be submitted (for many reasons)
  
  It depends on the version, but in most cases the MLNX-OFED will contain the features that exists in the upstream kernel code.
  ```

  

- MLNX-OFED包含了支持RDMA的软件包，包括用户空间和内核空间的代码，并且安装完后会升级Mellanox网卡的固件

- 重新编译MLNX-OFED时需要内核源码

- 加载RDMA驱动：/etc/init.d/openibd start

- RDMA需要锁定内存（不会被内核交换出去），默认情况下每个非root用户允许锁定64KB的内存，建议修改*/etc/security/limits.conf* 中这个值

- 





