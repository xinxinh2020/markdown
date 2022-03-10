



### 原理

在集群中部署一个 ValidatingWebhook，拦截特定访问 API Server 的请求.

webhookconfiguration: 

```yaml
Name:         loft
Namespace:    
Labels:       app.kubernetes.io/managed-by=Helm
Annotations:  meta.helm.sh/release-name: loft
              meta.helm.sh/release-namespace: loft
API Version:  admissionregistration.k8s.io/v1
Kind:         ValidatingWebhookConfiguration
Webhooks:
  Admission Review Versions:
    v1
    v1beta1
  Client Config:
    Ca Bundle:  ...
    Service:
      Name:        loft-webhook
      Namespace:   loft
      Path:        /validate
      Port:        443
  Failure Policy:  Fail
  Match Policy:    Equivalent
  Name:            validate.loft.sh
  Rules:
    API Groups:
      storage.loft.sh
    API Versions:
      v1
    Operations:
      CREATE
      UPDATE
    Resources:
      *
    Scope:          *
```

只对 storage.loft.sh 的 API Groups 的请求进行拦截？
