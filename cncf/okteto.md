#  源码解析

## 目录架构

### ~/.okteto

- okteto.log : 日志

### ~/.okteto/[namespace]/cli

- okteto.pid : 存 okteto cli 的 pid
- syncthing.info : 存放 Syncthing 对象 (pkg/syncthing/syncthing.go:70) 的一些信息
- syncthing.pid : 存放 syncthing 的 pid 信息，如果 okteto up 的时候检测到上一次操作的 syncthing 因为某种原因没能退出，则会通过发送一个 SIGINT 信号终结它
- okteto.state: 用来记录状态(cmd/up/state.go:25)，如 activating/starting/attaching/pulling/startingSync/synchronizing/ready 

# README

https://github.com/okteto/okteto
## 简介

传统的开发方式是通过 CI 任务或者 docker build/redeploy来进行，工作流太重，减低了开发速度，通过 okteto 可以实现直接在容器里开发，从而加速了 k8s 应用开发的工作流，在本地写完代码， okteto 会检测代码的变化并立刻更新 k8s 应用

## 原理

当运行 okteto up 命令时，k8s 的 deployment 会被替换为一个开发容器，这个开发容器包含了开发工具（maven、jdk、npm、python等），开发容器可以是任意 docker 镜像，开发容器继承原 deployment 的 secret，configmap,volume,等配置。除此之外， okteto up 还会：

- 创建一个双向的文件同步服务来同步本地文件系统和开发容器
- 使用 SSH 进行本地和远程的端口转发
- 提供一个交互式的终端，让你可以构建，测试和运行应用

以上通过配置一个 yaml manifest 实现

结果就是本地 IDE 和工具可以直接把远程集群当做一个本地的文件系统/环境。你只需要在本地的 IDE 编写你的代码，当你保存代码时，代码的改动会马上同步到开发容器里，你的应用会马上被更新（实际上需要应用有热加载机制，如果没有的话，可能还是需要重启一下的）。好处就是：不再需要制作新的镜像，也不需要去修改 kubernetes 的 manifest 文件并 apply 到集群里。



# docs

## CLI

### Okteto Manifest - okteto.yml

用来描述开发容器



## Cloud

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

