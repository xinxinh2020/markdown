[TOC]



## Jenkins

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/jenkins --set jenkinsUser=admin,jenkinsPassword=coding123
```

注意：需要一个 8G 以上的pv，密码长度必须大于 6 位数

deployment 监听端口：8080，8443

## ElasticSearch

```
helm install elasticsearch  elastic/elasticsearch --kubeconfig devcloud-config -n elasticsearch --set replicas=1,sysctlVmMaxMapCount=262144
```

注意，需要一个 30G 以上的 PV， 宿主机的 sysctl vm.max.map.count 要大于或等于262144

构建：./gradlew localDistro
