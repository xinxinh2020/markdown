[TOC]

## 总体架构

https://rancher.com/docs/rancher/v2.5/en/overview/architecture/

架构图：

![Architecture](images/rancher/rancher-architecture-rancher-api-server.svg)

交互流程：

![Rancher Components](images/rancher/rancher-architecture-cluster-controller.svg)

### Cluster Controllers

每个 downstream 集群上都有一个 agent，它会和 Rancher server 上对应的 cluster controller 保持一个 tunnel。

cluster controller 的作用：

- watch downstream 集群的资源变化
- Brings the current state of the downstream cluster to the desired state
- 为集群和项目配置访问控制策略
- Provisions clusters by calling the required Docker machine drivers and Kubernetes engines, such as RKE and GKE

### Authorized Cluster Endpoint

authorized cluster endpoint 允许用户直接连接到 downstream 的 k8s api server，只支持 RKE。

kube-api-auth 微服务用于 authorized cluster endpoint 的用户认证，它是一个 webhook。

authorized cluster endpoint 可用于 rancher server 和 downstream cluster 距离很远的场景或者 rancher server 挂了的场景

## 安装

### 在单个节点上使用 Docker 安装

#### 使用默认证书

使用以下命令在一个 docker 容器中部署 Rancher

```sh
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher
```

以上命令 Rancher 会生成一个默认的自签名证书。 



#### 使用生成的自签名证书

```sh
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
  -v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
  -v /<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
  --privileged \
  rancher/rancher:latest
```

参考文章：https://blog.csdn.net/yjw123456/article/details/115145409
```sh
docker run -d --privileged --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -e NO_PROXY="localhost,127.0.0.1,0.0.0.0,10.0.0.0/8,192.168.1.0/24" \
  -v /root/certs:/container/certs \
  -e SSL_CERT_DIR="/container/certs" \
  rancher/rancher:latest
```

