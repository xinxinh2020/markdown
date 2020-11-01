[TOC]

## 基础知识

istio是通过k8s namespace中是否带有`istio-injection=enabled` 标签来决定在这个namespace部署应用的时候要不要注入Envoy sidecar的。

### 安装 kali 仪表盘

```shell
kubectl apply -f samples/addons
while ! kubectl wait --for=condition=available --timeout=600s deployment/kiali -n istio-system; do sleep 1; done
```

通过 kali service的NortPort来访问 kali dashboard，如：http://9.135.94.198:20001/kiali/

默认账号：admin/admin



## 常用命令

```shell
istioctl analyze # 分析某个命名空间的资源是否有问题
istioctl dashboard kiali # 查看dashboard
```

## bookinfo示例程序

应用地址：https://raw.githubusercontent.com/istio/istio/release-1.7/samples/bookinfo/platform/kube/bookinfo.yaml

```shell
kubectl create ns demo
kubectl label namespace demo istio-injection=enabled # 启用istio
# kubectl apply -n -f samples/bookinfo/networking/bookinfo-gateway.yaml # 安装istio gateway(已集成到helm里)
```

判断环境是否提供外部的LB:

```shell
kubectl get svc istio-ingressgateway -n istio-system # 如果EXTERNAL-IP为空，则表明不提供
```

如果环境不提供外部的LB,则使用下面命令来设置ingress的port:

```shell
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')
```

设置防火墙规则，允许TCP流量进入 ingressgateway 的服务端口：

```shell
export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o jsonpath='{.items[0].status.hostIP}')
```

设置 GATEWAY_URL：

```shell
export GATEWAY_URL=$INGRESS_HOST:$INGRESS_PORT
```

访问bookinfo地址：http://9.135.94.198/productpage

### page

`./python productpage.py 9080`

python应用

### ratings 服务

启动命令：`node /opt/microservices/ratings.js 9080`

用来给图书评级，返回的是星星，不适合演示

### details 服务

/opt/microservices

`ruby details.rb 9080`

是个rb程序

### reviews 服务

启动命令：`/opt/ibm/java/jre/bin/java -javaagent:/opt/ibm/wlp/bin/tools/ws-javaagent.jar -Djava.awt.headless=true -Djdk.attach.allowAttach`

是个 java 服务。但是它有3个版本