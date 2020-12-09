

[TOC]



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

