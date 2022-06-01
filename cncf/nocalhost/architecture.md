

[TOC]

## 代码架构设计

Nocalhost 的核心组件是命令行工具 nhctl，IDE 插件（vscode, jetbrains）通过调用 nhctl 来对 K8S 集群中的工作负载进行操作（如进入开发模式，退出开发模式等）：

![image-20220601195359980](/Users/xinxinhuang/Workplace/markdown/cncf/nocalhost/architecture/image-20220601195359980.png)



### nhctl

构建 nhctl 二进制文件可以参考 [Using the Code Base]() , 本节主要介绍 nhctl 的源代码结构。

nhctl 基于 go 语言的 [cobra](https://github.com/spf13/cobra) 框架开发， 程序入口位于：https://github.com/nocalhost/nocalhost/blob/43d53217849acec221a6bf09f7ab3b99d1023e7f/cmd/nhctl/nhctl.go ，每个 nhctl 命令则对于 `cmds` 包下的一个文件。

nhctl 中有两个关键的数据结构，[Application](https://github.com/nocalhost/nocalhost/blob/43d53217849acec221a6bf09f7ab3b99d1023e7f/internal/nhctl/app/application.go) 和  [Controller](https://github.com/nocalhost/nocalhost/blob/43d53217849acec221a6bf09f7ab3b99d1023e7f/internal/nhctl/controller/service.go) 。Application 用于执行对 Nocalhost 应用的操作（如部署，删除等）, Controller 用于执行对 K8s 工作负载的操作（如进入开发模式，退出开发模式等）。

#### dev start

`dev start` 命令用于让指定的工作负载进入开发模式，主要代码逻辑位于 Controller  的 `ReplaceImage` 方法中，它会根据特定工作负载类型对应的 DevModeAction 提供的信息来对该工作负载执行进入开发模式的操作。对于 k8s 内置的工作负载类型(deployment/statefulset/daemonset/job/pod), nhctl 已经内置了对应的 DevModeAction, 对于 CRD 类型的工作负载，需要用户配置相关的 DevModeAction， DevModeAction 的配置及使用方法参考：https://nocalhost.dev/docs/guides/develop-service-crd-en.

`ReplaceImage` 的主要步骤为：

- 根据 DevModeAction 中定义的 ScalePatches ，在进入开发前执行相关的 Patch，如将副本数缩减为 1 等
- 根据 DevModeAction 中定义的 PodTemplatePath，找到工作负载的 PodTemplate
- 对工作负载的 PodTemplate 进行以下修改
  - 将业务容器镜像替换为开发容器镜像
  - 禁用开发容器的探针
  - 修改开发容器的 workDir
  - 添加运行文件同步服务的 sidecar 容器
  - 添加开发容器和 sidecar 容器共享的 Volume，用于存放同步过来的源代码
  - 添加持久化卷（optional）

#### dev end

#### dev config

#### vpn

#### daemon

### vscode 插件

### jetbrains 插件





