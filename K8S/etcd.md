

[TOC]

### 参数项

```shell
--initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 # 初始化集群的节点url,仅在引导集群初始化时有效，初始化后该参数不再需要，可去掉。参数中的url需要匹配每个节点的initial-advertise-peer-urls参数
--listen-client-urls http://10.0.1.10:2379,http://127.0.0.1:2379 # 监听客户端的请求
--initial-cluster-token etcd-cluster-1 # 用于生成一个集群IP来标识集群（可选）
--advertise-client-urls http://10.0.1.10:2379 # 提供给其他成员的url和远程客户端
--client-cert-auth # 让etcd验证每个进来的https请求
--trusted-ca-file=<path> # 受信任的CA，与上一步配合使用，客户端访问的时候需要提供证书，如curl --cacert /path/to/ca.crt --cert /path/to/client.crt --key /path/to/client.key 
--peer-cert-file=<path> # peer之间通信的证书
--cert-file=/path/to/server.crt
--key-file=/path/to/server.key  # etcd服务器的证书，使用自签名证书时，客户端访问时要提供ca.crt，如：curl --cacert /path/to/ca.crt https://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -v
```



### 集群搭建

#### Static

在启动之前，我们已经知道集群成员、他们的地址和集群的大小，因此可以通过设置initial-cluster标志来使用脱机引导配置。每台机器将获得以下环境变量或命令行:

```shell
# env
ETCD_INITIAL_CLUSTER="infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380"
ETCD_INITIAL_CLUSTER_STATE=new

# or command line
--initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
--initial-cluster-state new
```

etcd监听列表-客户端url以接受客户端流量。





### Design of runtime reconfiguration

运行时重新配置是分布式系统中最困难和最容易出错的特性之一，特别是在etcd这样的基于共识的系统中。



#### Two phase config changes keep the cluster safe

在etcd中，出于安全考虑，每次运行时重新配置都必须经过两个阶段。例如，要添加成员，首先通知集群新配置，然后启动新成员。

**Phase 1 - Inform cluster of new configuration**

要将一个成员添加到etcd集群中，需要调用一个API来请求将一个新成员添加到集群中。这是向现有集群添加新成员的惟一方法。当集群同意配置更改时，API调用返回。

**Phase 2 - Start new member**

将新的etcd成员加入到现有集群中，指定正确的initial-cluster并将initial-cluster-state设置为existing。当成员启动时，它将首先与现有集群联系，并验证当前集群配置是否符合在initial-cluster中指定的预期配置。当新成员成功启动时，集群已经达到了预期的配置。

通过将流程划分为两个独立的阶段，用户必须明确集群成员关系的变化。这实际上给了用户更多的灵活性，使事情更容易推理。例如，如果试图在etcd集群中添加与现有成员具有相同ID的新成员，则在第一阶段中操作将立即失败，而不会影响正在运行的集群。类似的保护以防止了错误地添加新成员。如果一个新的etcd成员试图在集群接受配置更改之前加入集群，集群将不会接受它。

如果没有围绕集群成员关系的显式工作流，etcd将容易受到意外的集群成员关系更改的影响。例如，如果etcd在诸如systemd之类的init系统下运行，etcd将在通过成员关系API删除后重新启动，并在启动时尝试重新加入集群。每次通过API删除一个成员时，这个循环都会继续，而systemd设置为在失败(这是意料之外的)后重新启动etcd。我们期望运行时重新配置是一个不常见的操作。我们决定保持它的显式和用户驱动，以确保配置安全，并保持集群始终在显式控制下平稳运行



#### Permanent loss of quorum requires new cluster

如果一个集群永久性地失去了大多数成员，需要从旧的数据目录启动一个新的集群，以恢复以前的状态。强制从现有集群中删除失败的成员来恢复是完全可能的。但是，我们决定不支持这个方法，因为它绕过了通常的一致同意提交阶段，这是不安全的。如果要删除的成员实际上并没有死，或者通过同一集群中的不同成员强制删除，etcd将以具有相同clusterID的发散集群结束。这是非常危险的，而且以后很难调试/修复。

通过正确的部署，永久性的多数派损失的可能性是非常低的。但这是一个非常严重的问题，值得特别关注。

#### Do not use public discovery service for runtime reconfiguration

公共发现服务应该只用于引导集群。要将成员加入现有集群，请使用runtime reconfiguration API。发现服务是为在云环境中引导etcd集群而设计的，在此之前，所有成员的IP地址都是未知的。成功引导集群之后，所有成员的IP地址都是已知的。从技术上讲，应该不再需要发现服务。

使用公共发现服务似乎是进行运行时重新配置的一种方便的方法，毕竟发现服务已经拥有了所有的集群配置信息。但是依赖公共发现服务带来的问题是:

1. 它为集群的整个生命周期引入了外部依赖，而不仅仅是引导时间。如果集群和公共发现服务之间存在网络问题，集群将受到影响。
2. 公共发现服务必须在其生命周期中反映集群的正确运行时配置。它必须提供安全机制来避免错误的操作，这是很困难的。
3. 公共发现服务必须保持数万个集群配置。我们的公共发现服务后端还没有为该工作负载做好准备。

要获得支持运行时重新配置的发现服务，最好的选择是构建一个私有的发现服务。



### Transport security model

https://github.com/coreos/docs/blob/master/os/generate-self-signed-certificates.md

要启动和运行，首先要有一个CA证书和一个成员的签名密钥对。建议为集群中的每个成员创建并签署一个新的密钥对。

证书类型：

- **client certificate**：用于通过服务器对客户端进行身份验证，如`etcdctl`, `etcd proxy`, 或 `docker` 客户端
- **server certificate**：由服务器使用，并由客户端验证服务器身份
- **peer certificate**：当etcd集群成员相互通信时使用

#### 证书生成

```shell
# 保存默认配置
mkdir ~/cfssl
cd ~/cfssl
cfssl print-defaults config > ca-config.json
cfssl print-defaults csr > ca-csr.json


```

