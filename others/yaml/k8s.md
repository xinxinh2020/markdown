[TOC]

## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: centos-01
spec:
  replicas: 1
  selector:
    matchLabels:
      app: centos-01
  template:
    metadata:
      labels:
        app: centos-01
    spec:
      containers:
      - name: details
        image: centos:latest
        imagePullPolicy: IfNotPresent
        command: ['/bin/sh', '-c', 'tail -f /dev/null']
```



## PVC

### tke

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-01
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: cbs
  volumeMode: Filesystem
```

## Pod

### resources limit

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
        cpu: 1
      requests:
        memory: "100Mi"
        cpu: 1
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

### liveness

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

### Volume

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reviews
spec:
  replicas: 1
  selector:
    matchLabels:
      app: reviews
  template:
    metadata:
      labels:
        app: reviews
    spec:
      containers:
        - image: codingcorp-docker.pkg.coding.net/nocalhost/bookinfo/reviews:latest
          imagePullPolicy: Always
          name: reviews
          ports:
            - containerPort: 9080
              protocol: TCP
          volumeMounts:
            - mountPath: /tmp
              name: tmp
            - mountPath: /opt/tmp
              name: work-dir-vol
            - mountPath: /opt/workdir
              name: pvc-vol-01
      volumes:
        - emptyDir: {}
          name: tmp
        - emptyDir: {}
          name: work-dir-vol
        - persistentVolumeClaim:
            claimName: pvc-01
            readOnly: false
          name: pvc-vol-01
```



### templates

```yaml
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: web
  namespace: nh30diml
  labels:
    app.kubernetes.io/managed-by: nocalhost
  annotations:
    dev.nocalhost/application-name: bookinfo-manifest
    dev.nocalhost/application-namespace: nh30diml
    kubectl.kubernetes.io/last-applied-configuration: >
      {"apiVersion":"apps/v1","kind":"StatefulSet","metadata":{"annotations":{"dev.nocalhost/application-name":"bookinfo-manifest","dev.nocalhost/application-namespace":"nh30diml"},"labels":{"app.kubernetes.io/managed-by":"nocalhost"},"name":"web","namespace":"nh30diml"},"spec":{"replicas":3,"selector":{"matchLabels":{"app":"web"}},"serviceName":"nginx","template":{"metadata":{"labels":{"app":"web"}},"spec":{"containers":[{"image":"lowyard/nginx-slim:0.8","name":"nginx","ports":[{"containerPort":80,"name":"web"}],"volumeMounts":[{"mountPath":"/usr/share/nginx/html","name":"www"}]}],"terminationGracePeriodSeconds":10}},"volumeClaimTemplates":[{"metadata":{"name":"www"},"spec":{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"10Gi"}}}}]}}
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: 'lowyard/nginx-slim:0.8'
          ports:
            - name: web
              containerPort: 80
              protocol: TCP
          volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: www
        creationTimestamp: null
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 10Gi
        volumeMode: Filesystem
      status:
        phase: Pending
  serviceName: nginx
```



## Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-sa-sample
  annotations:
    kubernetes.io/service-account.name: "sa-name"
type: kubernetes.io/service-account-token
data:
  extra: YmFyCg==
```

