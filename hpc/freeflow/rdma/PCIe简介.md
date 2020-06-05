[TOC]

## PCIe简介

PCIe用于系统中不同模块之间的通信。网络适配器需要和CPU和内存进行通信，这意味着为了处理网络流量，通过PCIe进行通信的不同设备需要配置好。当把网络适配器连接到PCIe上，它会自动协商网络适配器于PCIe之间支持的最大的容量。	

任何加载进来的PCI设备都有特定的属性，一些属性对性能来说是关键。设备的PCIe属性是通过在系统容量和设备容量之间协商出来的，因此选择两边都支持的最高值。下面是相关属性的一些解释，如何验证它们以及它们在性能方面的影响。

### PCIe宽度

PCIe宽度决定了可以被设备并行地用于通信的PCIe lane(通道)数量，宽度是以xA作为标记的，A是通道是数量(比如x8表示8通道)。Mellanox适配器支持x8和x16配置，取决于具体类型。为了验证PCIe的宽度，可以使用lspci命令。

下面我们以安装在PCI 02.00.0地址()上的Mellanox适配器为例：

```shell
[root@power27 test-classes]#  lspci -s 02:00.0 -vvv | grep Width
                LnkCap: Port #8, Speed 8GT/s, Width x8, ASPM L0s, Exit Latency L0s unlimited, L1 unlimited
                LnkSta: Speed 8GT/s, Width x8, TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-
```

如上所示，LnkCap显示了设备的能力(Capabilities)，LnkSta显示了实际PCIe设备的属性。



### PCIe速度

这决定了可能的PCIe事务的数量。速度的单位是GT/s(billion transactions per second)。和宽度一起，PCIe最大带宽的计算方法为：速度*宽度。查看PCI速度可以通过lspci命令：

```shell
[root@power27 test-classes]#  lspci -s 02:00.0 -vvv | grep Speed
                LnkCap: Port #8, Speed 8GT/s, Width x8, ASPM L0s, Exit Latency L0s unlimited, L1 unlimited
                LnkSta: Speed 8GT/s, Width x8, TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-
```

PCI速度被标记为generation（代），2.5GT/s称为gen1,5GT/s称为gen2,8GT/s称为gen3。大部分Mellanox产品支持所有generation。查看PCI的generation同样可以通过lspci命令：

```shell
[root@power27 test-classes]#  lspci -s 02:00.0 -vvv | grep Gen
                        [V0] Vendor specific: PCIe Gen3 x8
```

不同代之间除了速度之外的一个主要不同点是包的编码开销。第1和第2代，通过PCIe发送的每个包有20%的头部开销，而第3代优化到1.5%。

### PCIe 最大负载（Max  Payload Size）

PCIe最大负载决定了一个PCIe包最大的大小或PCIe MTU（和网络协议类似）。这意味着超过这个大小的PCIe事务会被分解为多个PCIe MTU大小的包。这个参数是由系统设置的，取决于芯片组的架构（如x86_64，Power8或ARM等）。可以通过如下方式查看PCI的最大负载：

```shell
[root@power27 test-classes]#  lspci -s 02:00.0 -vvv | grep DevCtl: -C 2
                DevCap: MaxPayload 512 bytes, PhantFunc 0, Latency L0s <64ns, L1 unlimited
                        ExtTag- AttnBtn- AttnInd- PwrInd- RBE+ FLReset- SlotPowerLimit 116.000W
                DevCtl: Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
                        RlxdOrd- ExtTag- PhantFunc- AuxPwr- NoSnoop-
                        MaxPayload 256 bytes, MaxReadReq 512 bytes
```



### PCIe最大读请求（Max Read Request）

它决定了PCIe允许的最大读请求。一个PCIe设备通常会保持追踪正在进行的读请求的数量，因为需要为响应准备缓冲区。通过以下方式可以查看PCI的最大读请求：

```shell
[root@power27 test-classes]#  lspci -s 02:00.0 -vvv | grep MaxReadReq
                        MaxPayload 256 bytes, MaxReadReq 512 bytes
```

和其他参数不同的是，PCIe的最大读请求可以在运行期修改，修改方式如下：

```shell
# setpci -s 04:00.0 68.w
5936
# setpci -s 04:00.0 68.w=2936  #第一个数字是最大读请求的选项：0 - 128B, 1 - 256B, 2 - 512B, 3 - 1024B, 4 - 2048B and 5 - 4096B
# lspci -s 04:00.0 -vvv | grep MaxReadReq
MaxPayload 256 bytes, MaxReadReq 512 bytes
```



### 计算PCIe的限制

如上所述，PCIe的Capabilities会影响网络适配器的性能。理解PCIe带来的带宽限制是有好处的。下面是理论上的计算和一些例子。

理论上最大的PCIe带宽是PCIe的速度乘以宽度。考虑到错误修正协议（error correction protocol），我们从这个值减去1Gb/s。头部开销取决于PCIe编码（第1代和第2代为1/5，第3代为2/130）。因此最大带宽计算公式可以总结为：速度 * 宽度 * (1 - 编码开销) - 1Gb/s。

例如一个第3代的PCIe设备，宽度为x8，它的最大带宽为：8G * 8 * (1 - 2/130) - 1Gb/s = 62Gb/s

需要注意的是，PCIe事务包括了网络包的有效负载和头，如果计算网络流量的限速时需要把网络头的开销考虑进去。

参考资料：https://community.mellanox.com/s/article/understanding-pcie-configuration-for-maximum-performance