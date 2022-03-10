

```shell
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: nocalhost-container-critical
value: 1000000
description: "Nocalhost"
```

