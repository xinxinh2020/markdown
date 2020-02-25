

[TOC]



## Pod

### 资源限制

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```



## PV

### 本地

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
  labels:
    name: pv001
spec:
  hostPath:
    path: /k8s/pv/pv01
  accessModes: ["ReadWriteMany","ReadWriteOnce","ReadOnlyMany"]
  capacity:
    storage: 1Gi
```

### nfs

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
  labels:
    name: pv001
spec:
  nfs:
    path: /data/volumes/v1
    server: k8s-node2
  accessModes: ["ReadWriteMany","ReadWriteOnce","ReadOnlyMany"]
  capacity:
    storage: 1Gi
```

