[TOC]



## 基础

Telepresence 通过在 Kubernetes 集群中运行的 pod 中部署双向网络代理来工作。这个 pod 将数据从 Kubernetes 环境（例如 TCP 连接、环境变量、卷）代理到本地进程。此代理可以拦截用于服务的流量并将其重新路由到本地副本，为进一步（本地）开发做好准备。



## 拦截

https://www.telepresence.io/docs/latest/howtos/intercepts/

### 连接集群

使用 `telepresence connect` 可以连接 current-context 指向的集群，连接集群后，会自动创建一个 `ambassador`命名空间，在该命名空间下创建一个 `traffic-manager` deployment，打通本地和集群的 VPN 连接，连接上集群以后，可以直接访问集群的 DNS，ping Pod 的 IP 地址：

```shell
curl -ik https://kubernetes.default
HTTP/1.1 403 Forbidden
Cache-Control: no-cache, private
Content-Type: application/json
X-Content-Type-Options: nosniff
```



### 拦截服务

命令：

```shell
telepresence intercept <service-name> --port <local-port>[:<remote-port>] --env-file <path-to-env-file>
```

该命令会在工作负载中添加一个 `traffic-agent`的容器，该容器会拦截指定端口的流量，并把它转发到本地端口。

### 登录

在登录后，进行拦截的时候，telepresence 会和 Ambassador Cloud 进行通信，来生成一个 preview Url, telepresence 只会把来自这个 preview Url 的请求路由到本地环境，原来的请求路径还是会被路由到原来集群中的服务。

登录后进行拦截的时候，会要求填写一些信息，如 ingress 的地址，来生成 preview Url

## 多人协作

原理：https://www.getambassador.io/docs/telepresence/latest/concepts/context-prop

多人协作需要命令行工具登录 [Ambassador Cloud](https://app.getambassador.io/) , 在使用 `telepresence intercept` 的时候，会要求填入 ingress 地址等信息，telepresence 会把这些信息发给 Ambassador Cloud, Ambassador Cloud 会生成一个 preview Url，通过这个 preview Url 访问的时候请求才会被路由到本地，原来的请求路径还是会被路由到原来集群中的服务。

使用 preview Url 访问的时候，会要求用户登录 Ambassador Cloud （因为该 URL 会被 agent 拦截，没有登陆 Cloud 的请求是不能通过的），登录后该 URL 会显示在 Ambassador Cloud 界面的列表中，点击的时候，会被加上 header (猜测)，该 header 通过 ingress 最终透传（借助 tracing 等技术）到 agent，agent 根据 header 信息来对请求进行路由。



### 技术细节

环境变量通过导出到本地文件之后再注入到容器中

卷通过挂在到本地目录再把本地目录挂到容器中