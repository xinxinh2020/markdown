#  源码解析

## 文件信息

### ~/.okteto

- okteto.log : 日志
- id_rsa_okteto.pup: ssh 公钥
- id_rsa_okteto: ssh 私钥

### ~/.okteto/[namespace]/[deployment]/

- okteto.pid : 存 okteto cli 的 pid
- syncthing.info : 存放 Syncthing 对象 (pkg/syncthing/syncthing.go:70) 的一些信息
- syncthing.pid : 存放 syncthing 的 pid 信息，如果 okteto up 的时候检测到上一次操作的 syncthing 因为某种原因没能退出，则会通过发送一个 SIGINT 信号终结它
- okteto.state: 用来记录状态(cmd/up/state.go:25)，如 activating/starting/attaching/pulling/startingSync/synchronizing/ready 

## Lables

```shell
dev.okteto.com=true  # 该 pod 正在开发中
interactive.dev.okteto.com=api # up 的时候打在 pod 上，api 为 deployment 的名字
dev.okteto.com/not-allowed   # 用于 namespace，用于告诉 okteto 不允许在该 namespace 上进行操作
```

## Annotation

```shell
dev.okteto.com/version=1.0 # 版本号，up 的时候会给 deployment 打上
```


# README

https://github.com/okteto/okteto
## 概览

Kubernetes 已经让部署大规模的应用到云上变得非常容易，但是开发实践没能跟上应用部署的发展速度。


如今，大多数开发人员尝试在本地运行部分基础架构，或者只是通过 CI 作业或 docker build/redeploy 周期直接在集群中进行集成测试。这行得通，但是整个工作流程很痛苦而且非常慢。

`okteto` 加快了 Kubernetes 应用的开发工作流。开发者在本地编写代码，okteto 可以检测到更改并立即更新对应的 Kubernetes 应用。

## 它是如何运作的

Okteto 允许你在容器内进行开发。当运行 `okteto up` 时，Kubernetes 的 deployment 会被一个开发容器替换，开发容器中包含了常用的开发工具（例如 maven 和 jdk ，或者 npm，python，go编译器，调试器等）。该开发容器可以是任何[docker 镜像]（https://okteto.com/docs/reference/development-environment）。开发容器继承了与原始 Kubernetes deployment 相同的 secret，configmap，volume 等配置。

除此之外， okteto up 还会：

- 创建一个双向的 [文件同步服务](https://okteto.com/docs/reference/file-synchronization) 来同步本地文件系统和开发容器的文件改动
- 使用 [SSH](https://okteto.com/docs/reference/ssh-server) 进行本地和远程的端口转发，因此你可以通过 `localhost` 来访问集群服务或连接一个远程 debugger
- 提供一个交互式的终端，让你可以构建，测试和运行应用

以上通过配置一个 yaml manifest 实现

结果就是本地 IDE 和工具可以直接把远程集群当做一个本地的文件系统/环境。你只需要在本地的 IDE 编写你的代码，当你保存代码时，代码的改动会马上同步到开发容器里，你的应用会马上被更新（实际上需要应用有热加载机制，如果没有的话，可能还是需要重启一下的）。好处就是：不再需要制作新的镜像，也不需要去修改 kubernetes 的 manifest 文件并 apply 到集群里。

![Okteto](docs/okteto-architecture.png)

# 文档

## Okteto CLI

### 命令

#### namespace

用来登录 Okteto Cloud， 并将云的 k8s kubeconfig 添加到本地的 ~/.kube/config，并将其设置为当前的 context。

#### up

该命令需要在服务的源码的目录下执行，目录下的 `okteto.yml` 用来配置该服务的一些开发参数:

```yaml
name: api
command: ["bash"]
forward:
  - 8080:8080
  - 9229:9229
sync:
  - .:/src
persistentVolume:
  enabled: false
```

使用 -f 可以指定外挂的 `okteto.yml`。

该命令会将 Pod 副本数置1，替换开发镜像，镜像可以在`okteto.yml`中指定，如果不指定，默认使用的是当前容器的镜像？？？？

开发容器是原 k8s 的拷贝，并附加以下功能：

- 启动一个文件同步服务，把 `sync` 中定义的目录同步到开发容器中去
- 根据 `forward` 中定义的端口规则进行转发
- 开启一个 terminal 直接连到容器中

##### 源码详解

替换镜像的时候还会修改容器的启动命令为 ` /var/okteto/bin/start.sh`，这个文件来自于 initContainer 向名为 okteto-bin 的 emptyDir Volume 拷贝的文件， 这个 container 的镜像是 okteto/bin:1.2.8

默认会开启远程模式，除非环境变量 OKTETO_EXECUTE_SSH 设置为 false

ssh 密钥一开始的时候会为用户生成一对，以后会复用这对密钥

okteto 会修改 ~/.ssh/config 中的配置，在该配置中追加如下配置：

```shell
Host api.okteto
  ForwardAgent yes
  HostName localhost
  Port 56132
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  IdentityFile "/Users/xinxinhuang/.okteto/id_rsa_okteto"
```

port-forward 到容器中 okteto-reomte 监听的 2222 端口

其他端口通过 ssh 转发: 8384,22000,8080,9229

#### build

从一个 Dockerfile 中构建一个镜像，并推到仓库中，用法类似于 `docker build`，okteto 提供的官方仓库是`registry.cloud.okteto.net`，每天每个账号有 15 个免费构建的配额。也可以将镜像推送到任意其他仓库中

#### pipeline

okteto-pipeline.yml demo:

```yaml
icon: https://apps.okteto.com/movies/icon.png
deploy:
  - okteto build -t okteto.dev/api:${OKTETO_GIT_COMMIT} api
  - okteto build -t okteto.dev/frontend:${OKTETO_GIT_COMMIT} frontend
  - helm upgrade --install movies chart --set tag=${OKTETO_GIT_COMMIT}
devs:
  - api/okteto.yml
  - frontend/okteto.yml
```

##### deploy

效果等同于点击 UI 界面上的 `deploy` 按钮，在执行该命令前需要先登录

##### destroy

效果等同于点击 UI 界面上的 `destroy` 按钮，在执行该命令前需要先登录

## Okteto Cloud

https://okteto.com/docs/cloud

[Okteto Cloud](https://cloud.okteto.com/#/?origin=docs) 提供了免费的 k8s namespaces，可以用来跑要开发的服务。

### 部署方法

#### 从 Git 部署

Git 仓库里需要先创建一个 pipeline 文件：

```yaml
icon: https://apps.okteto.com/movies/icon.png
deploy:
  - okteto build -t okteto.dev/api:${OKTETO_GIT_COMMIT} api
  - okteto build -t okteto.dev/frontend:${OKTETO_GIT_COMMIT} frontend
  - helm upgrade --install movies chart --set tag=${OKTETO_GIT_COMMIT}
devs:
  - api/okteto.yml
  - frontend/okteto.yml
```

- deploy: 部署应用的命令，一般是：helm, kubectl 和 okteto 的组合
- devs: git 仓库中每个服务的 okteto 配置，一般用来预定义一些配置，在 Develop 对话框中填入一些默认参数（可选）

## Reference

### okteto.yml

链接：https://okteto.com/docs/reference/manifest/index.html

okteto.yml : 

```yaml
name: vote
labels:
  app.kubernetes.io/part-of: vote
  app.kubernetes.io/component: api
image: python:3
command: ["python", "app.py"]
workdir: /usr/src/app
sync:
 - .:/usr/src/app
environment:
  - name=$USER
  - environment=development
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
forward:
  - 8080:80
  - 5432:postgres:5432
reverse:
  - 9000:9001
securityContext:
  runAsUser: 1000
  runAsGroup: 2000
  fsGroup: 3000
  capabilities:
    add:
    - SYS_PTRACE
```

- labels: 选中 deployment 的时候需要匹配的标签，需要匹配所有标签

## FAQ

- up 的时候，okteto 也会把 deployment 副本数置为 1