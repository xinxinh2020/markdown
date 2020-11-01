[TOC]

# Helm 基础

## 基础知识

Chart: A *Chart* is a Helm package

Repository: A *Repository* is the place where charts can be collected and shared

Release: A *Release* is an instance of a chart running in a Kubernetes cluster.

Hub: 集合了多个repo的资源

helm包中的Chart.yaml可以定义依赖:

```shell
cat >> Chart.yaml <<EOF
# 依赖关系
dependencies:
- name: greeter
  # 版本，如下匹配0.开头的所有版本
  version: 0.x.x
  # helm仓库地址，如同image镜像，helm也可推送到仓库，此仓库我们后续再配置
  repository: http://chartmuseum.app.zyl.io
  # 通过此条件判断是否依赖于此应用
  condition: greerer.enabled
  # 可为此应用添加一些标签
  tags:
    - http
EOF
```

## 依赖

 Chart中声明依赖的最佳实践：https://helm.sh/docs/chart_best_practices/dependencies/ 

尽可能使用https://的url，其次是http://的，  @？？

file://是一种特殊的场景，当charts被集成到一个固定的流水线上。

### Chart.yaml

Chart.yaml中可用的字段如下：

```yaml
apiVersion: The chart API version (required) v2: 只有helm3支持
name: The name of the chart (required)
version: A SemVer 2 version (required) chart包的版本号和这个字段必须相同
kubeVersion: A SemVer range of compatible Kubernetes versions (optional)
description: A single-sentence description of this project (optional)
type: The type of the chart (optional)
keywords:
  - A list of keywords about this project (optional)
home: The URL of this projects home page (optional)
sources:
  - A list of URLs to source code for this project (optional)
dependencies: # A list of the chart requirements (optional)
  - name: The name of the chart (nginx)
    version: The version of the chart ("1.2.3")
    repository: The repository URL ("https://example.com/charts") or alias ("@repo-name")
    condition: (optional) A yaml path that resolves to a boolean, used for enabling/disabling charts (e.g. subchart1.enabled )
    tags: # (optional)
      - Tags can be used to group charts for enabling/disabling together
    enabled: (optional) Enabled bool determines if chart should be loaded
    import-values: # (optional)
      - ImportValues holds the mapping of source values to parent key to be imported. Each item can be a string or pair of child/parent sublist items.
    alias: (optional) Alias to be used for the chart. Useful when you have to add the same chart multiple times
maintainers: # (optional)
  - name: The maintainers name (required for each maintainer)
    email: The maintainers email (optional for each maintainer)
    url: A URL for the maintainer (optional for each maintainer)
icon: A URL to an SVG or PNG image to be used as an icon (optional).
appVersion: The version of the app that this contains (optional). This needn't be SemVer.
deprecated: Whether this chart is deprecated (optional, boolean)
annotations:
  example: A list of annotations keyed by name (optional).
# Other fields will be silently ignored.
```

 #### apiVersion

v2仅helm3支持，v1和v2的区别：

- v1中的依赖定义在一个单独的requirements.yaml文件中，而v2定义在Chart.yaml的dependencies字段中。
- 增加了type字段，用来区分应用程序和库的chart

#### kubeVersion

支持的 kubernetes 版本

#### type

有两种类型：application和library。默认是application类型，library类型的chart是不能被安装的，不包含任何资源对象（即使有也会被忽略）

#### dependencies

依赖声明：

```yaml
dependencies:
  - name: apache
    version: 1.2.3
    repository: https://example.com/charts
  - name: mysql
    version: 3.2.1
    repository: https://another.example.com/charts
```

- name: chart名
- version: chart的版本
- repository: chart repo的url地址，必须提前使用`helm repo add`在本地添加好 。 也可以使用repo的名字而不用url，如下repo：

```sh
helm repo add fantastic-charts https://fantastic-charts.storage.googleapis.com
```

依赖可以声明如下：

```yaml
dependencies:
  - name: awesomeness
    version: 1.0.0
    repository: "@fantastic-charts"
```

定义好依赖以后，可以使用



## 部署安装

```shell
# 下载安装包
#https://get.helm.sh/helm-v3.1.0-linux-amd64.tar.gz
wget https://get.helm.sh/helm-v2.16.3-linux-amd64.tar.gz
tar zxvf helm-v2.16.3-linux-amd64.tar.gz
cd linux-amd64/
mv helm /usr/bin/
kubectl --namespace kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller --wait
# 有个tiller镜像可能下不了，要自己fq去下:gcr.io/kubernetes-helm/tiller:v2.16.3

```

## 常用命令

```shell
helm repo add stable https://kubernetes-charts.storage.googleapis.com/ # 添加一个名为stable的仓库
helm repo list # 列出已添加的repo
helm repo update # 更新repo
helm repo remove # 删除repo
helm search repo stable # 列出stable仓库中所有可用的chart，在本地查找即可，不需要连网
helm search repo # 列出本地已添加的仓库中的所有可用的chart
helm search hub # 列出hub中所有的chart
helm search hub # 列举hub中所有可用的chart
helm search hub wordpress # 列举所有公共可见的chart

helm pull [CHART] # 将chart包下载下来

helm dependency update # 更新依赖，把依赖下载到charts目录中，以tgz的形式

# helm install
helm install <chart-name> --generate-name # 安装mysql 																		
helm install foo foo-0.1.1.tgz # 本安装
helm install foo path/to/foo # 从指定目录安装
helm install foo https://example.com/charts/foo-1.2.3.tgz # 在线安装
# install 的一些重要参数
# -f指定yaml配置文件
# --set 指定配置项
# --wait 等待直到所有Pod就绪，PVC绑定好，service拥有一个IP地址...
# --timeout k8s命令超时的时间
# --debug 显示错误输出

helm show values stable/mysql # 显示该chart的可选配置项，可以把这些配置项保存在yaml文件中，修改yaml文件中的配置项，在安装的时候使用该yaml文件来做自定义配置
helm show chart stable/mysql # 显示chart的简介
helm show all stable/mysql # 显示chart的详情
helm show values stable/mariadb # 显示chart的配置信息

helm serve --address 0.0.0.0:8879 --repo-path /dcos/appstore/local-repo # 在本地启动一个 repo

helm get values <release-name> # 查看自定义配置项，即install时--set指定的配置项
helm get manifest -n <release-name> # 查看release的manifest
helm delete <release name> # 删除release
helm list # 列出已安装的release
helm uninstall <release-name> # 删除一个release --keep-history 保存历史记录
helm status <release-name> # 查看release的状态信息
helm update # 升级release的chart版本或配置信息，eg: helm upgrade -f panda.yaml happy-panda stable/mariadb
helm history [RELEASE] # 查看release的历史版本
helm rollback [RELEASE] [REVISION] # 回滚， eg: helm rollback happy-panda 1

helm create [CHART] # eg: helm create deis-workflow, 使用./deis-workflow目录创建一个chart
helm package [CHART] # 将做好的chart打包
										# --dependency-update : update dependencies from “requirements.yaml” to dir “charts/” before packaging
helm repo index . # Read the current directory and generate an index file based on the charts found
```



## template

内建对象：

- Release
  - Name
  - Namespace
  - IsUpgrade：是否是升级操作
  - IsInstall：是否是安装操作
  - Revision：install的时候是1，之后每次升级逐渐递增
  - Service：渲染模板的引擎，在helm中的值固定为helm
- Values：values.yaml或用户提供的配置文件，默认情况下 Values 为空
- Chart: Chart.yaml
- Files：访问chart中的非特殊文件（不能访问template）
- Capabilities：k8s集群提供的能力
- Template: 正在被执行的模板

## Chart制作

Helm 的资源文件使用了`text/template`，还使用`Sprig`这个模板函数库，此外还添加了2个特殊的模板函数：`include`和`required`。

include可以在一个template中引入其他template，如下,引入了一个叫mytpl的模板，并转为小写，然后加上引号：

```
value: {{ include "mytpl" . | lower | quote }}
```

required 函数可以声明一个特定的值为必须的，如果该值为空，模板渲染会失败。如下声明了 .Values.who 为必须的： 

```
value: {{ required "A valid .Values.who entry required!" .Values.who }}
```

Tpl 函数可以在一个templdate中把一个字符串当做一个template。

chart包中的charts目录包含了子chart



## Chart Hook

Chart hook的作用：

- 在安装chart前加载一个ConfigMap或Secret
- 在安装一个chart前，执行一个Job来备份一个数据库，在升级之后运行第二个Job来恢复数据
- 在删除一个release之前运行一个Job来优雅地退出

 pre-install hook在chart的模板渲染后执行，它有可能是个Job或Pod，也有可能是其他资源，资源的执行顺序可以通过权重来定义。

hook创建的资源目前没有作为release的一部分管理起来，所以当hook执行完之后，它们就留在那里，即使执行helm uninstall把release删除了，hook创建的资源也不会被删除（后续版本可能会加入这一功能）

### 编写一个hook

hook只是一些kubernetes的manifest文件，在metadata中带有一些特殊的annotations，如下：

```
annotations:
  "helm.sh/hook": post-install
```

它们是模板文件（放在templates目录下），所以可以使用所有模板特性，包括读取 `.Values`,`.Release`和`.Template` 

可以使用以下注解来指定一个hook的权重：

```
annotations:
  "helm.sh/hook-weight": "5"
```

helm会以升序执行这些hook。

可以用以下注解指定删除策略：

```
annotations:
  "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
```

有以下3个值可选：

- before-hook-creation ： 在一个新的hook执行之前删除上一个hook创建的资源（默认）
- hook-succeeded：hook成功执行完之后删除其所创建的资源
- hook-failed：如果hook执行失败，则删除其所创建的资源

可以指定一个或多个值，如果没有指定，则默认使用before-hook-creation



## 常用repo

gitlab: helm repo add gitlab https://charts.gitlab.io/



## helm应用安装

### Gitlab安装

https://docs.gitlab.com/charts/installation/deployment.html

参数设置：

域名设置：

```shell
--set global.hosts.domain=example.com
```

tls：

```shell
--set certmanager-issuer.email=652085787@qq.com
```

使用社区版本：

```shell
--set global.edition=ce
```

安装命令：

```sh
helm install gitlab-02 gitlab/gitlab \
--set global.hosts.domain=example.com \
--set global.edition=ce \
--set certmanager-issuer.email=652085787@qq.com
```

获取自动生成的root用户初始化密码：

```shell
kubectl get secret <name>-gitlab-initial-root-password -ojsonpath='{.data.password}' | base64 --decode ; echo
# SbkQ3KUUM4cDlbK78n0Y4GdulB7VcaAv9foOrxOIDYpQaOQVoT7jnn6ovVl1aEhU
```



# Helm Chart Releaser

https://github.com/marketplace/actions/helm-chart-releaser

在github上创建一个release，并把chart包上传到release中

下载：

```shell
wget https://github.com/helm/chart-releaser/releases/download/v1.0.0/chart-releaser_1.0.0_linux_amd64.tar.gz
tar zxvf chart-releaser_1.0.0_linux_amd64.tar.gz
cp cr /usr/local/bin/
cr -h
```

