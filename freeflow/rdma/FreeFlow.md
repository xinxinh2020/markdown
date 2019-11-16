[TOC]

## 步骤

### 启动FreeFlow Router

#### 运行FreeFlow Router（FFR）容器

   ```shell
   # docker run --name router1 --net host -e "FFR_NAME=router1" -e "LD_LIBRARY_PATH=/usr/lib/:/usr/local/lib/:/usr/lib64/" -v /sys/class/:/sys/class/ -v /freeflow:/freeflow -v /dev/:/dev/ --privileged -it -d ubuntu:14.04 /bin/bash
   # 官方是用ubuntu镜像，但ubuntu镜像在我们的环境上安装OFED时有点问题，以下使用CentOS7的镜像
   docker run --name router1 --net host -e "FFR_NAME=router1" -e "LD_LIBRARY_PATH=/usr/lib/:/usr/local/lib/:/usr/lib64/" -v /sys/class/:/sys/class/ -v /freeflow:/freeflow -v /dev/:/dev/  --privileged -it -d centos:7 /bin/bash
   # 把主机的/lib/modules目录拷进去，安装kernel-devel-3.10.0-957.el7.x86_64.rpm及其他依赖
   
   ```

#### 下载RDMA库和驱动（OFED）
 在[Mellanox官网](http://www.mellanox.com/page/products_dyn?product_family=26) 上下载驱动包：MLNX_OFED_LINUX-4.0-2.0.0.1-ubuntu14.04-x86_64.tgz( CentOS的需要用CentOS相应的版本)

#### 在FFR容器中安装OFED驱动
    将驱动包拷贝到主机的/FreeFlow目录下并解压，进入容器中，进入解压后的目录，执行mlnxofedinstall程序进行驱动安装：

  ```shell
  # 进入Docker容器
  [root@power27 freeflow]# docker exec -it router1 bash
  # 安装依赖
  # yum install perl python-devel redhat-rpm-config rpm-build make pciutils numactl-libs gtk2 atk cairo gcc-gfortran tcsh lsof libnl3 libmnl ethtool tcl tk gcc-c++
  [root@power27 freeflow]# cd /freeflow/MLNX_OFED_LINUX-4.6-1.0.1.1-rhel7.6-x86_64
  [root@power27 MLNX_OFED_LINUX-4.6-1.0.1.1-rhel7.6-x86_64]# ./mlnxofedinstall 
  
  ```
#### 构建FFR并运行FFR
```shell
# 构建FFR
./build-router.sh # 直接运行会报错，ffrouter.cpp文件有点问题，需要把文件最后的函数实现放到最前面去
# 运行FFR
./router router1
```



### 宿主机安装weave

```shell
curl -L git.io/weave -o /usr/local/bin/weave
chmod a+x /usr/local/bin/weave
weave launch
docker network ls # 确保网络列表中有weave
    eval $(weave env)
```



### 运行FFL容器 

```shell
docker run --name node1 --net weave -e "FFR_NAME=router1" -e "FFR_ID=10" -e "LD_LIBRARY_PATH=/usr/lib" -e --ipc=container:router1 -v /sys/class/:/sys/class/ -v /freeflow:/freeflow -v /dev/:/dev/ --privileged --device=/dev/infiniband/uverbs0 --device=/dev/infiniband/rdma_cm -it -d 036f23f274f8 /bin/bash

cd /freeflow/Freeflow-master/libmempool
make
make install

yum -y install automake libtool
# 安装libnl

cd ../libraries/libibverbs-1.2.1mlnx1/
./autogen.sh
# 将Makefile.in中的AM_CFLAGS = -g -Wall -Werror改成AM_CFLAGS = -g -Wall
./configure LIBS=-lrt --prefix=/usr/ --libdir=/usr/lib/ --sysconfdir=/etc/  LIBS=-lmempool
make
make install

cd ../libmlx4-1.2.1mlnx1/
./autogen.sh
# 将Makefile.in中的AM_CFLAGS = -g -Wall -Werror改成AM_CFLAGS = -g -Wall
./configure --prefix=/usr/ --libdir=/usr/lib/ --sysconfdir=/etc/
make
make install

cd ../librdmacm-1.1.0mlnx/
./autogen.sh
./configure --prefix=/usr/ --libdir=/usr/lib/ --sysconfdir=/etc/
make
make install

```





### 验证

#### 修改FFR路由表



```shell
ib_send_lat
### FreeFlow ###
context->qp_table_mask=2047
mlx4: Warning: BlueFlame available, but failed to mmap() BlueFlame page.

************************************
* Waiting for client to connect... *
************************************
```

