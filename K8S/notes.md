[TOC]





## API版本

### 版本改动

#### 1.16.x

- 放弃了部分API支持,apps/v1beta1和apps/v1beta2不再使用，用apps/v1代替；
- extensions/v1beta1下的daemonsets, deployments, replicasets，使用apps/v1代替
- extensions/v1beta1下的networkpolicies用networking.k8s.io/v1代替
- extensions/v1beta1下的podsecuritypolicies 使用policy/v1beta1代替
- 

```shell
https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.16.md#deprecations-and-removals
 
API
The following APIs are no longer served by default:
All resources under apps/v1beta1 and apps/v1beta2 - use apps/v1 instead
daemonsets, deployments, replicasets resources under extensions/v1beta1 - use apps/v1 instead
networkpolicies resources under extensions/v1beta1 - use networking.k8s.io/v1 instead
podsecuritypolicies resources under extensions/v1beta1 - use policy/v1beta1 instead
Serving these resources can be temporarily re-enabled using the --runtime-config apiserver flag.
```



