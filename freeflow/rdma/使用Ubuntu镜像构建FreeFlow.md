

[TOC]



# 步骤

## 安装ubuntu 14.04的虚拟机



## 安装Docker环境

```sh
apt-get install docker.io
docker pull ubuntu:14.04
```

## 构建FFR

### 运行ubuntu容器

```shell
docker run --name router1 --net host -e "FFR_NAME=router1" -e "LD_LIBRARY_PATH=/usr/lib/:/usr/local/lib/:/usr/lib64/" -v /sys/class/:/sys/class/ -v /freeflow:/freeflow -v /dev/:/dev/ --privileged -it -d ubuntu:14.04 /bin/bash
```



### 下载RDMA库驱动

 在[Mellanox官网](http://www.mellanox.com/page/products_dyn?product_family=26) 上下载驱动包：MLNX_OFED_LINUX-4.0-2.0.0.1-ubuntu14.04-x86_64.tgz

### 安装驱动

```shell
cd /freeflow/
tar zxvf MLNX_OFED_LINUX-4.0-2.0.0.1-ubuntu14.04-x86_64.tgz
cd MLNX_OFED_LINUX-4.0-2.0.0.1-ubuntu14.04-x86_64
# 注意把apt源配好	
./mlnxofedinstall
```

### 导出镜像到测试环境

```shell
# 在虚拟机上
docker commit -m "ofed installed" router1 ubuntu_ofed
docker save ubuntu_ofed > ubuntu_ofed
# 在测试环境(power27)
docker load < ubuntu_ofed
```

### 运行ubuntu_ofed

```shell
docker run --name router01 --net host -e "FFR_NAME=router1" -e "LD_LIBRARY_PATH=/usr/lib/:/usr/local/lib/:/usr/lib64/" -v /sys/class/:/sys/class/ -v /freeflow:/freeflow -v /dev/:/dev/ --privileged -it -d ubuntu_ofed /bin/bash
```



### 安装FFR

```shell
docker exec -it router01 bash
cd /freeflow/Freeflow-master/
# 修改硬编码的路由表
./build-router.sh
./router router1
```



## 构建FFL

```shell
docker run --name node1 --net weave -e "FFR_NAME=router1" -e "FFR_ID=10" -e "LD_LIBRARY_PATH=/usr/lib" --ipc=container:router1 -v /sys/class/:/sys/class/ -v /freeflow:/freeflow -v /dev/:/dev/ --privileged --device=/dev/infiniband/uverbs0 --device=/dev/infiniband/rdma_cm -it -d ubuntu_ofed /bin/bash
docker exec -it node1 bash

cd /freeflow/Freeflow-master/libmempool
make
make install

# 安装libnl	
cd /freeflow/libnl-3.2.25
./configure --prefix=/usr --sysconfdir=/etc --disable-static
make
make install

cd /freeflow/Freeflow-master/libraries/libibverbs-1.2.1mlnx1/
./autogen.sh
# 将Makefile.in中的AM_CFLAGS = -g -Wall -Werror改成AM_CFLAGS = -g -Wall
./configure --prefix=/usr/ --libdir=/usr/lib/ --sysconfdir=/etc/ LIBS=-lmempool
# 在Makefile中的LIBS后面加上-lrt
make
make install

```



## 将FFR和FFL做成镜像

```shell
docker commit -m "ffl installed" node1 ubuntu_ffl
docker commit -m "ffr installed" router01  ubuntu_ffr
```



## 验证

### 配置FFR的路由

### 启动FFR

```shell
# 不同机器的FFR_NAME不能相同
docker run --name router1 --net host -e "FFR_NAME=router1" -e "LD_LIBRARY_PATH=/usr/lib/:/usr/local/lib/:/usr/lib64/" -v /sys/class/:/sys/class/ -v /public/hxx/freeflow:/freeflow -v /dev/:/dev/ --privileged -it -d ubuntu_ffr /bin/bash
```



### 启动两个FFL容器

```shell
docker run --name node1 --net weave -e "FFR_NAME=router1" -e "FFR_ID=10" -e "LD_LIBRARY_PATH=/usr/lib"  --ipc=container:router1 -v /sys/class/:/sys/class/ -v /public/hxx/freeflow:/freeflow -v /dev/:/dev/ --privileged --device=/dev/infiniband/uverbs0 --device=/dev/infiniband/rdma_cm -it -d ubuntu_ffl /bin/bash
docker run --name node2 --net weave -e "FFR_NAME=router1" -e "FFR_ID=11" -e "LD_LIBRARY_PATH=/usr/lib"  --ipc=container:router1 -v /sys/class/:/sys/class/ -v /public/hxx/freeflow:/freeflow -v /dev/:/dev/ --privileged --device=/dev/infiniband/uverbs0 --device=/dev/infiniband/rdma_cm -it -d ubuntu_ffl /bin/bash
```



### 进入FFL容器使用perftest进行测试

```shell
# in node1
root@c3082b435dd0:/# ib_send_lat -D 20
### FreeFlow ###
context->qp_table_mask=2047
mlx4: Warning: BlueFlame available, but failed to mmap() BlueFlame page.

************************************
* Waiting for client to connect... *
************************************

# in node2 , 10.32.0.1的node的IP地址
root@31cc35b70358:/# ib_send_lat 10.32.0.1 -D 20
### FreeFlow ###
context->qp_table_mask=2047
mlx4: Warning: BlueFlame available, but failed to mmap() BlueFlame page.
@@@ ibv_exp_cmd_query_device @@@
[INFO] Succeed to mount shm ctrlshm-qp8
---------------------------------------------------------------------------------------
                    Send Latency Test
 Dual-port       : OFF          Device         : mlx4_0
 Number of qps   : 1            Transport type : IB
 Connection type : RC           Using SRQ      : OFF
 TX depth        : 1
 Mtu             : 2048[B]
 Link type       : IB
 Max inline data : 236[B]
 rdma_cm QPs     : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0x0e QPN 0x0344 PSN 0x62cc34
 remote address: LID 0x0e QPN 0x0345 PSN 0x89e08e
---------------------------------------------------------------------------------------
 #bytes        #iterations       t_avg[usec]    tps average
Conflicting CPU frequency values detected: 1293.811000 != 1200.024000. CPU Frequency is not max.
Conflicting CPU frequency values detected: 1204.467000 != 3237.145000. CPU Frequency is not max.
 2             1289984            3.88           128993.46
---------------------------------------------------------------------------------------


```



