[TOC]



## 常用命令



```shell
docker run -it -v /home/dock/Downloads:/usr/Downloads ubuntu64 /bin/bash # 冒号前为宿主机目录，必须为绝对路径，冒号后为镜像内挂载的路径，默认权限为rw

# Swarm
docker swarm init --advertise-addr <manager ip> # 创建一个swarm集群
docker swarm join  --token SWMTKN-1-2k314zcj87ca9it9d2bvzuhawe4buj7e3jetox9kfiwaz4o9la-37vgf4wlopp0om2f7sbgi6424  192.168.23.129:2377 # 把当前节点加入到sware中
docker node ls # 查看集群中节点的加入状况
docker swarm leave # 退出swarm集群管理
docker history  c3b52b2a0265  --no-trunc # 查看镜像的构建过程

# 导出镜像
docker save docker.elastic.co/kibana/kibana:7.6.2 > kibana.tar
docker save docker.elastic.co/kibana/kibana:7.6.2 -o kibana.tar
# 导入镜像
docker load < kibana.tar
docker load -i kibana.tar
```

