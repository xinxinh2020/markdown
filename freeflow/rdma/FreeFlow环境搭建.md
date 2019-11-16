[TOC]

### 启动FFR

```shell
# in HOST1
docker run --name router1 --net host -e "FFR_NAME=router1" -e "LD_LIBRARY_PATH=/usr/lib/:/usr/local/lib/:/usr/lib64/" -v /sys/class/:/sys/class/ -v /public/hxx/freeflow:/freeflow -v /dev/:/dev/ --privileged -it -d ubuntu_ffr /bin/bash

# in HOST2
docker run --name router2 --net host -e "FFR_NAME=router2" -e "LD_LIBRARY_PATH=/usr/lib/:/usr/local/lib/:/usr/lib64/" -v /sys/class/:/sys/class/ -v /public/hxx/freeflow:/freeflow -v /dev/:/dev/ --privileged -it -d ubuntu_ffr /bin/bash
```



### 启动两个FFL容器

```shell
# in Host1
docker run --name node1 --net weave -e "FFR_NAME=router1" -e "FFR_ID=10" -e "LD_LIBRARY_PATH=/usr/lib"  --ipc=container:router1 -v /sys/class/:/sys/class/ -v /public/hxx/freeflow:/freeflow -v /dev/:/dev/ --privileged --device=/dev/infiniband/uverbs0 --device=/dev/infiniband/rdma_cm -it -d ubuntu_ffl /bin/bash
docker run --name node2 --net weave -e "FFR_NAME=router1" -e "FFR_ID=11" -e "LD_LIBRARY_PATH=/usr/lib"  --ipc=container:router1 -v /sys/class/:/sys/class/ -v /public/hxx/freeflow:/freeflow -v /dev/:/dev/ --privileged --device=/dev/infiniband/uverbs0 --device=/dev/infiniband/rdma_cm -it -d ubuntu_ffl /bin/bash

# in Host2
docker run --name node1 --net weave -e "FFR_NAME=router2" -e "FFR_ID=10" -e "LD_LIBRARY_PATH=/usr/lib"  --ipc=container:router2 -v /sys/class/:/sys/class/ -v /public/hxx/freeflow:/freeflow -v /dev/:/dev/ --privileged --device=/dev/infiniband/uverbs0 --device=/dev/infiniband/rdma_cm -it -d ubuntu_ffl /bin/bash
docker run --name node2 --net weave -e "FFR_NAME=router2" -e "FFR_ID=11" -e "LD_LIBRARY_PATH=/usr/lib"  --ipc=container:router2 -v /sys/class/:/sys/class/ -v /public/hxx/freeflow:/freeflow -v /dev/:/dev/ --privileged --device=/dev/infiniband/uverbs0 --device=/dev/infiniband/rdma_cm -it -d ubuntu_ffl /bin/bash
```



### 配置FFR

```shell
# 修改/public/hxx/freeflow/Freeflow-master/ffrouter目录下的ffrouter.h 和 ffrouter.cpp中的路由映射表
# 在宿主机的共享目录上编译即可
cd /public/hxx/freeflow/Freeflow-master/
./build-router.sh
# in HOST1
docker exec -it router1 bash
cd /freeflow/Freeflow-master/ffrouter
./router router1

# in HOST2
docker exec -it router2 bash
cd /freeflow/Freeflow-master/ffrouter
./router router2
```

