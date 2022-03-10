## 设计文档
### IDE 插件
IDE 插件在 Workloads 的同级树节点增加一个 CustomResources 的节点，该节点下渲染三级子节点，分别为 `Group`, `Kind(Version)` 以及对应 CRD 类型的资源对象列表：
```
- [namespace]
---- [application]
------- [Workloads]
------- [CustomResources]
---------- storage.loft.sh
---------- apps.kruise.io  <- Group
------------- CloneSet(v1alpha1)   <- Kind(Version) 
---------------- productpages <- clonesets.apps.kruise.io 类型的资源对象
---------------- reviews
------------- Daemonset(v1alpha1) 
------------- Sidecarset(v1alpha1) 
----- -- [Networks]
----- -- [Configurations]
```
其中：
- `Group`, `Kind(Version)` 调用 `nhctl get crd-list` 获取，需要 cluster admin 权限
- 资源对象列表的获取及对资源对象的所有操作和 `Workloads` 下的资源对象操作保持一致(调用 nhctl 命令传递对应的资源对象类型即可)
- 资源对象列表的获取不需要 cluster admin 权限

#### 权限问题
CRD 类型的获取需要 cluster admin 权限，CRD 非集群级的资源对象列表获取不需要 cluster admin 权限，但是插件获取不到 CRD 类型就不知道有哪些类型的 CRD 资源对象可以列出来，目前想到的一个方案是插件提供让用户手动输入 CRD 类型的功能，从而让只有 namespace 级别权限的用户也可以开发 CRD，具体需要：
- 在 `CustomResources` 节点上增加一个 `+` 按钮，用来收到添加 CRD 类型
- 添加 CRD 类型时需要输入以下信息
    - Group,Version, Kind, Resource
    - 是否为 Namespace 级资源(这个可以暂时忽略，目前不支持开发集群级的 CRD 对象，因为不知道展示在哪里）
- 插件可以通过调 nhctl 命令检查用户输入的 CRD 类型是否存在及是否有权限
- 添加完以后插件需要把这个 CRD 类型信息记录起来，该信息和 kubeconfig 关联，同 kubeconfig 不同 namespace 可共享，vscode 和 jetbrains 可约定保存在相同的地方（kubeconfig 也可以考虑两个插件共享）
- 用户自己添加的 CRD 类型支持删除操作（可选，不支持删除也没太大问题）

### nhctl 
nhctl 本次改动比较大的是去除了大部分针对具体资源类型的操作(如 Deployment,StatefulSet 等)，通过 `ResourceType` 获取到通用的 `Unstructured` 来进行操作。Daemon 将 `SharedInformerFactory` 统一改成了 `DynamicSharedInformerFactory`，内置类型也是通过 `DynamicSharedInformerFactory` 来获取。

对 `Unstructured 的操作` ，可以通过 `DevModeAction` 来控制。内置类型(包括 K8s 标准的工作负载类型如 Deployment/StatefulSet 等，以及常见的开源 CRD，如 kruise 项目提供的 CRD) 通过内置的 `DevModeAction` 来控制，用户可以在 `~/.nh/nhctl/config` 文件中手动添加特定 CRD 类型相关的 `DevModeAction` 来让 Nocalhost 拥有开发该 CRD 对象的能力。配置如下：
```shell
crdDevModeActions:
- crdType: statefulsets.v1beta1.apps.kruise.io
  devModeAction:
    podTemplatePath: /spec/template/
    create: false
    scalePatches:
    - type: json
      patch: '[{"op":"replace","path":"/spec/replicas","value":1}]'
```
其中：
- crdType: 为 `resource.version.group` 的形式，即 GVR 反过来写，指定该 `DevModeAction` 对应的 CRD 类型
- devModeAction: 用于控制 Nocalhost 让指定 CRD 对象进入 `DevMode` 时的行为
    - podTemplatePath: 指定 Pod 模板定义所在的路径，Nocalhost 会修改该模板，如将容器镜像修改成开发镜像，添加 sidecar 容器等
    
    - scalePatches: 定义在进入 `DevMode` 时，将 CRD 对象的副本数缩成 1 的 patch。该 Patch 的自由度比较高，并不局限于将 CRD 对象的副本数缩成 1 ，这里可以把它当作类似 helm 的 `preHook`, 实现一些 hack 操作，如果不需要将 CRD 对象的副本数缩成 1，可以不配
	
	- create: 指明 CRD 对象在进入 `DevMode` 时，是否需要创建一个新的 Deployment，在该 Deployment 上进行开发
	    - 如果为 `true`，Nocalhost 会使用在 `podTemplatePath` 中找到的 Pod 模板定义来创建一个 Deployment，`DevMode` 下的所有操作都在改 Deployment 中进行，可以参考内置的 `CronJob` 类型的 `DevModeAction`：
			```go
	    	CronJobDevModeAction = base.DevModeAction{
				ScalePatches: []base.PatchItem{{
					Patch: `{"spec":{"suspend": true}}`,
					Type:  "strategic",
				}},
				PodTemplatePath: "/spec/jobTemplate/spec/template",
    			Create:          true,
	    	}
			```
	    - 如果为 `false`，Nocalhost 会直接修改 CRD 对象的 Pod 模板定义，`DevMode` 下的所有操作都在改 CRD 对象上进行，可以参考内置的默认 `DevModeAction`：
			```go
	    	DefaultDevModeAction = base.DevModeAction{
				ScalePatches: []base.PatchItem{{
					Patch: `[{"op":"replace","path":"/spec/replicas","value":1}]`,
					Type:  "json",
				}},
				PodTemplatePath: "/spec/template",
	    	}
			```



剩余问题：

- 多个 Version 的 CRD 资源
- CRD 资源的状态显示

## Todo

- [x]  非 admin 的 kubeconfig get deployment 得不到数据 
- [x] 多个 Version 的 CRD 支持
- [ ] 判断哪个版本是 Preferred Version 
- [x] daemon 启动时需要 init 多次 searcher
- [x] crd 没有 description
- [x] get 有时候会列不出数据
- [ ] 列举 Pod 的使用使用 Controller-reference
- [ ] duplicate 模式的工作负载名字太长
- [x] crd 不能使用 stategic 类型的 patch（统一使用 json 类型的）
- [x] crd-list 没有权限不要报错
- [ ] auth check

## 插件端支持
Kind, resourceName deployments
插件端需要在树结构中展示 CRD 资源列表，大体的展示形式如下：
```
- [namespace]
---- [application]
------- [Workloads]
------- [CustomResources]
---------- storage.loft.sh
---------- apps.kruise.io  <- Group
------------- CloneSet   <- Kind RE
---------------- productpages <- clonesets.apps.kruise.io 类型的资源对象
---------------- reviews
------------- Daemonset
------------- Sidecarset
----- -- [Networks]
----- -- [Configurations]
```

具体说明：
- 在 Workloads 下增加一个 CustomResources 的列表项
- 使用 `nhctl get crd-list -o yaml` 获取数据，数据结构如下：

```
- info:
    Group: storage.loft.sh
    Kind: AccessKey
    Namespaced: false
    Resource: accesskeys
    Version: v1
- info:
    Group: config.kiosk.sh
    Kind: Account
    Namespaced: false
    Resource: accounts
    Version: v1alpha1
- info:
    Group: apps.kruise.io
    Kind: AdvancedCronJob
    Namespaced: true
    Resource: advancedcronjobs
    Version: v1alpha1
```
- 集群级（Namespaced = false）的暂时忽略
- 将获取到的 Group 显示在 CustomResources 下
- 将获取到的 Kind 显示在 Group 下 （可能会有多个 Kind 位于同个 Group 下）
- 使用 `nhctl get [Resource.Version.Group] -a [appName] -n [namespace] -o json` 获取特定类型的 CRD 资源对象，将资源对象列表显示在 [Kind] 下。数据结构如下:

``` yaml
nhctl get clonesets.v1alpha1.apps.kruise.io -a bookinfo -n nocalhost-test -o yaml
- info:
    apiVersion: apps.kruise.io/v1alpha1
    kind: CloneSet
    metadata:
        annotations:
            dev.nocalhost/application-name: bookinfo
            dev.nocalhost/application-namespace: nocalhost-test
            dev.nocalhost/origin-workload-definition: '{"apiVersion":"apps.kruise.io/v1alpha1","kind":"CloneSet","metadata":{"annotations":{"dev.nocalhost/application-name":"bookinfo","dev.nocalhost/application-namespace":"nocalhost-test"},"generation":11,"labels":{"app":"sample","app.kubernetes.io/managed-by":"nocalhost"},"name":"sample","namespace":"nocalhost-test","uid":"44049638-d9ae-46a2-9f4e-7dd411b054e7"},"spec":{"replicas":5,"revisionHistoryLimit":10,"scaleStrategy":{},"selector":{"matchLabels":{"app":"sample"}},"template":{"metadata":{"creationTimestamp":null,"labels":{"app":"sample"}},"spec":{"containers":[{"image":"nginx:alpine","imagePullPolicy":"Always","name":"nginx","resources":{},"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File"}],"dnsPolicy":"ClusterFirst","restartPolicy":"Always","schedulerName":"default-scheduler","securityContext":{},"terminationGracePeriodSeconds":30}},"updateStrategy":{"maxSurge":0,"maxUnavailable":"20%","partition":0,"type":"ReCreate"}}}'
            kubectl.kubernetes.io/last-applied-configuration: |
                {"apiVersion":"apps.kruise.io/v1alpha1","kind":"CloneSet","metadata":{"annotations":{"dev.nocalhost/application-name":"bookinfo","dev.nocalhost/application-namespace":"nocalhost-test"},"labels":{"app":"sample","app.kubernetes.io/managed-by":"nocalhost"},"name":"sample","namespace":"nocalhost-test"},"spec":{"replicas":5,"selector":{"matchLabels":{"app":"sample"}},"template":{"metadata":{"labels":{"app":"sample"}},"spec":{"containers":[{"image":"nginx:alpine","name":"nginx"}]}}}}
        creationTimestamp: "2021-12-07T07:21:21Z"
        generation: 14
        labels:
            app: sample
            app.kubernetes.io/managed-by: nocalhost
    spec:
        replicas: 5
        revisionHistoryLimit: 10
        scaleStrategy: {}
        selector:
            matchLabels:
                app: sample
        template:
            metadata:
                creationTimestamp: null
                labels:
                    app: sample
            spec:
                containers:
                    - image: nginx:alpine
                      imagePullPolicy: Always
                      name: nginx
                      resources: {}
                      terminationMessagePath: /dev/termination-log
                      terminationMessagePolicy: File
                dnsPolicy: ClusterFirst
                restartPolicy: Always
                schedulerName: default-scheduler
                securityContext: {}
                terminationGracePeriodSeconds: 30
        updateStrategy:
            maxSurge: 0
            maxUnavailable: 20%
            partition: 0
            type: ReCreate
    status:
        availableReplicas: 5
        collisionCount: 0
        currentRevision: sample-d4d4fb5bd
        labelSelector: app=sample
        observedGeneration: 14
        readyReplicas: 5
        replicas: 5
        updateRevision: sample-d4d4fb5bd
        updatedReadyReplicas: 5
        updatedReplicas: 5
```
- 资源对象上支持和 Workloads 一样的操作（如：进入开发，端口转发等），调用 nhctl 的命令和 Workloads 类似，只需要将 -t 参数改为 `[Resource].[Version].[Group]`



## 前期调研
###目前 Nocalhost 进入开发模式对工作负载的修改整理
#### Deployments/StatefulSets
1. 记录原始的 `DeploymentSpec` 或者 `StatefulSetSpec`
2. 将副本数缩成 1
3. 修改 `PodTemplateSpec`
    1. 修改目标容器名为 `nocalhost-dev`
    2. 设置目标容器的启动命令为 `/bin/sh -c tail -f /dev/null`
    3. 设置目标容器的镜像为开发镜像
    4. 设置目标容器的 `workDir`
    5. 设置目标容器的 `imagePullPolicy` 为 `Always`
    6. 给目标容器添加环境变量
    7. 给目标容器设置 `resources` (optional)
    2. 添加卷
        - nocalhost-syncthing: EmptyDir 类型的 Volume，挂载到容器中的 `/var/syncthing` 目录
        - nocalhost-syncthing-secret: 挂载 syncthing 的 secrets 到容器中的 `/var/syncthing/secret` 目录
        - nocalhost-shared-volume: EmptyDir 类型的 Volume，挂载到容器中的 `workDir`
        - `persist-volume-%d`: 持久化卷
        - 以上卷都同时挂在到 `nocalhost-dev` 容器 和 `nocalhost-sidecar` 容器中，实际上 syncthing 相关的卷并不需要挂载到 `nocalhost-dev` 容器，持久化卷不需要挂在到 `nocalhost-sidecar` 容器
    3. Pod 级别的 `SecurityContext` 设为空
    10. 添加 `nocalhost-sidecar` 容器
    11. 禁用所有容器的所有探针及 `SecurityContext`
    12. 设置 `priorityClassName` (optional)
4. patch 其它自定义属性

#### DaemonSets
1. 将 DaemonSets 的副本数缩成 0 （通过设置一个不存在的 nodeName）
2. 生成一个 Deployment ， Deployment 的 `LabelSelector` 和 `PodTemplateSpec` 和 DaemonSets 相同
3. 修改 `PodTemplateSpec` (和上述相同)
3. patch 其它自定义属性

#### Job
- 生成一个 Job (实际上这里也可以生成一个 Deployment，区别不大)
- 修改 `PodTemplateSpec`
- patch 其它自定义属性

#### Cronjobs
- 记录 Cronjobs 的 suspend 状态（用于退出开发模式恢复）
- 将 Cronjobs 挂起
- 生成一个 Job，Job 的 `LabelSelector` 和 `PodTemplateSpec` 和 CronJobs 相同 （实际上这里应该也可以生成个 Deployment）
- 修改 `PodTemplateSpec` (和上述相同)
- patch 其它自定义属性

#### Pod 
- 记录原始 Pod 对象
- 删除原始 Pod 对象
- 生成一个 Pod，Pod 属性和原始 Pod 相同
- 修改 `PodTemplateSpec` (和上述相同)
- patch 其它自定义属性

### 流程总结
通用步骤可以抽象成：
1. 记录原始的 Spec 定义
2. 根据具体情况决定是否需要在新建出来的 deployment 上进行操作
2. 将副本数缩成 1
3. 修改 `PodTemplateSpec`
4. patch 其它自定义属性

需要知道的 CRD 信息：
- 是在原来的工作负载上操作还是新建一个 deployment 来操作
- `PodTemplateSpec` 的位置
- 副本数缩成 1 的方法

对于类似 CronJobs 这种，感觉不用支持，支持它管理的 Jobs 就好，如果用户想测试 Cronjobs 调度 Jobs 的逻辑，也不会用到 Nocalhost，如果想测试的是 Cronjobs 创建出来的 Jobs 里跑的业务，用 Nocalhost 去开发 Jobs 就可以了。

以上三者都可以通过让用户在 annotation 里面指定的方式来实现，对于后两者：
- `PodTemplateSpec` 只需要指定类似：`spec.template.spec` 的路径
- 可以让用户提供一段 patch 代码来 DIY 副本缩成 1 的方式，如：
    - 对于 deployment，patch 为：`'{"spec":{"replicas": 1}}'`
    - 对于 daemonset, patch 为：`'{"spec":{"template:": {"spec": {"nodeName": "nocalhost.unreachable"}}}}'`

### 初步规划
- 将现有的 deployments/statefulsets/jobs 的开发改成使用上述通用流程来进行，提高