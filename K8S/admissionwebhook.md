

# 动态Admission Control

https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/

除了编译的admission插件，还可以作为插件开发并且在运行期作为webhook运行，以下介绍如何构建，配置，使用和监控admission webhook。

## admission webhook是啥？

admission webhook是http的回调，它会接受admission请求并对其进行处理。你可以定义两种类型的admission webhook：validating（验证） admission webhook和mutating（修改） admission webhook。mutating admission webhook会先被调用，并且可以修改发给api server的对象。validating admission webhook用于根据自定义的策略来拒绝请求。



## admission webhook实战

Admission webhook是k8s集群控制面板的一个关键部分，应该谨慎地编写和部署它们。

### 前置准备

- 确保集版本不低于v1.16（admissionregistration.k8s.io/v1），或v1.9（admissionregistration.k8s.io/v1beta1）

- 确保启用了 `MutatingAdmissionWebhook`和`ValidatingAdmissionWebhook`admission controller。v1.10及以上版本，以下admission controller是默认启用的(可以参考[apiserver的参数](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/#options))：

  ```sh
  NamespaceLifecycle, LimitRanger, ServiceAccount, TaintNodesByCondition, Priority, DefaultTolerationSeconds, DefaultStorageClass,StorageObjectInUseProtection, PersistentVolumeClaimResize, RuntimeClass, CertificateApproval, CertificateSigning, CertificateSubjectRestriction, DefaultIngressClass,MutatingAdmissionWebhook,ValidatingAdmissionWebhook, ResourceQuota
  ```

- 确保启用了 `admissionregistration.k8s.io/v1` 或 `admissionregistration.k8s.io/v1beta1 ` API

### 编写一个admission webhook server

可以参考一个实现例子：[admission webhook server](https://github.com/kubernetes/kubernetes/blob/v1.13.0/test/images/webhook/main.go)，这个webhook处理apiserver发送的AdmissionReview`请求，处理后返回一个和接收时相等的`AdmissionReview` 对象。

发给webhook的数据可以参考webhook request. webhook发出去的数据可以参考webhook response.

在上述例子中，`ClientAuth`字段为空，表明它使用默认值`NoClientCert`。这意味着webhook server不验证client的身份。

### 部署 admission webhook service

demo中的webhook server以deployment的方式被部署在集群中，并为其创建了一个service，详见[代码](https://github.com/kubernetes/kubernetes/blob/v1.15.0/test/e2e/apimachinery/webhook.go#L301)

### 配置 admission webhook

你可以通过`ValidatingWebhookConfiguration`或者`MutatingWebhookConfiguration`动态地配置什么资源订阅admission webhook

webhooks.rules.scope字段有3个取值：

- Cluster：集群范围
- Namespaces：命名空间范围
- "*" ：没有范围限制

webhook调用默认的超时时间是10秒（v1, v1beta1是30秒），从1.14开始支持设置超时时间，推荐把超时时间设置成一个比较小的值。如果webhook调用超时，request会根据webhook的失败策略进行处理。

当一个apiserver接收到一个匹配rules中任意一个规则的request时，apiserver会发送一个admissionReview 请求给clientConfig中指定的webhook

## Webhook request和response



### response

response的数据放在patch和patchType字段，



## Webhook configuration

要注册admission webhook，需要先创建MutatingWebhookConfiguration，ValidatingWebhookConfiguration API对象。

每个configuration可以包含1个或多个webhook。

每个webhook定义了以下事情。

### rules

每个webhook必须指定一个rules列表用于觉得一个request是否应该被发送到webhook。每个rule指定一个或多个操作，apiGroups,apiVersions,resource和resource scope：

- operations：指定一个或多个要匹配的操作，可取的值有：CREATE,UPDATE,DELETE,CONNECT,*
- apiGroups：指定1个或多个要匹配的API groups。""表示是core API组，"*"表示匹配所有API group
- apiVersions：指定1个或多个要匹配的api versions。
- resources：指定一个或多个要匹配的资源：

  - "*"表示匹配所有资源，但不包括子资源

  - "*/*"表示匹配所有资源和子资源

  - "pods/*"表示匹配pods的所有子资源

  - "*/status"表示匹配所有status子资源
- scope：Cluster,Namespaced,*



### objectSelector

在v1.15及以上版本，webhook可以基于对象的标签来限制哪些request会被拦截。

### namespaceSelector

匹配特定namespace（带有某些label的namespace）的资源或Namespace资源本身



### matchPolicy

v1.15及以上支持：

- Exact：意味着一个request只有精确匹配某条rule时才会被拦截
- Equivalent：意味着如果request的api group/version不匹配，把api group/version修改成rule中列出的值通过的话，就会被拦截

Equivalent是推荐的做法，确保当资源的api version升级后还能正常被拦截

### 调用webhook

clientConfig指定了webhook的配置。

webhook可以通过一个url或一个服务引用来调用，并且可以包括一个CA来验证TLS连接。

#### URL

url指定了webhook的位置。url的形式为：`scheme://host:port/path`

host应该是一个运行在集群外部的域名或IP地址。运行在集群内的应该用Service的方式。

#### Service

指定service的namespace和name,默认的端口是443,默认的路径的/

#### Side Effects

#### Timeouts

#### reinvocationPolicy

#### failurePolicy

- Ignore：失败后这个请求可以继续
- Fail：失败后这个请求会被拒绝

