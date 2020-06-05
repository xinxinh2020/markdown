# 验证RDMA内核模块是否已加载

centos7下：

```shell
[root@power27 docs]# /etc/init.d/openibd status

  HCA driver loaded

Configured IPoIB devices:
ib0

Currently active IPoIB devices:
ib0
Configured Mellanox EN devices:

Currently active Mellanox devices:
ib0

The following OFED modules are loaded:

  rdma_ucm
  rdma_cm
  ib_ipoib
  mlx4_core
  mlx4_ib
  mlx4_en
  mlx5_core
  mlx5_ib
  ib_uverbs
  ib_umad
  ib_ucm
  ib_cm
  ib_core
  mlxfw
  mlx5_fpga_tools
```

ibv_devices是一个包含在libibverbs-utils.rpm包里的工具，用于显示本机上的RDMA设备：

```shell
[root@power27 docs]# ibv_devices
    device                 node GUID
    ------              ----------------
    mlx4_0              e41d2d030050e830
```

ibv_devinfo也是libibverbs-utils.rpm包中的一个工具，它会打开一个设备查询设备的属性，通过它可以验证用户空间和内核空间的RMDA栈是否能够一起正常运作：

```shell
[root@power27 docs]# ibv_devinfo -d mlx4_0
hca_id: mlx4_0
        transport:                      InfiniBand (0)
        fw_ver:                         2.42.5000
        node_guid:                      e41d:2d03:0050:e830
        sys_image_guid:                 e41d:2d03:0050:e833
        vendor_id:                      0x02c9
        vendor_part_id:                 4099
        hw_ver:                         0x1
        board_id:                       MT_1100120019
        phys_port_cnt:                  1
        Device ports:
                port:   1
                        state:                  PORT_ACTIVE (4)
                        max_mtu:                4096 (5)
                        active_mtu:             4096 (5)
                        sm_lid:                 1
                        port_lid:               12
                        port_lmc:               0x00
                        link_layer:             InfiniBand
```

至少要有一个端口的状态是PORT_ACTIVE，才能说明RDMA相关组件已经正常运行起来。

使用bv_x_pingpong测试RDMA设备的流量发送功能：

```shell
# 在服务端
ibv_rc_pingpong -g 0 -d mlx4_0 -i 1
  local address:  LID 0x000c, QPN 0x000a19, PSN 0xf31d1e, GID fe80::e41d:2d03:50:e831
  remote address: LID 0x000e, QPN 0x000491, PSN 0xfefc9e, GID fe80::e41d:2d03:50:e801
8192000 bytes in 0.01 seconds = 11821.07 Mbit/sec
1000 iters in 0.01 seconds = 5.54 usec/iter

#在客户端 192.168.10.27是服务端的地址
ibv_rc_pingpong -g 0 -d mlx4_0 -i 1 192.168.10.27
  local address:  LID 0x000e, QPN 0x000491, PSN 0xfefc9e, GID fe80::e41d:2d03:50:e801
  remote address: LID 0x000c, QPN 0x000a19, PSN 0xf31d1e, GID fe80::e41d:2d03:50:e831
8192000 bytes in 0.01 seconds = 11797.66 Mbit/sec
1000 iters in 0.01 seconds = 5.55 usec/iter
```

