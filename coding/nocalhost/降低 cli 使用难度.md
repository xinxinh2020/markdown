



## 如何确定集群，namespace  

Okteto: okteto login/namespace 修改 ~/.kube/config 的 context

DevSpace: 集群直接使用 ~/.kube/config 对应的集群， namespace 使用 devspace use namespace my-namespace 切换，如果没有该 namespace，会创建一个

Nocalhost: 可以从 localhost 的 profile 和 ~/.kube/config 同时拿

## 如何识别应用名，以及源码关联的服务 

Okteto: 没有应用名，服务名写在源码目录下的 okteto.yml

DevSpace: 在源码目录执行 `devspace init` 会进入一个交互界面，有以下步骤：

-  如何部署应用（helm,kubectl,kustomize）
- 如何构建本服务镜像 - > 把镜像推到哪

生成一个 devspace.yaml，通过 imageSeletor 来找到运行指定镜像的工作负载进行开发

Nocalhost: 从集群的 meta 拿应用列表和服务列表让用户选择

## 开发配置

从 meta 拿，如果缺少配置，则提示让用户在 ui 进行配置，或选择一个 开发镜像

## 问题

### 同一个源码目录关联了多个工作负载

Devspace: 可以一个命令开发多个工作负载，替换工作负载除了 image，其他操作都是通过 patches 实现的。terminal 也是自己指定某个 pod，及命令来进入 terminal

