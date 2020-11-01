



## node port

```yaml
apiVersion: v1
kind: Service
metadata:
  name: productpage
  labels:
    app: productpage
    service: productpage
spec:
  type: NodePort
  ports:
  - port: 9080
    name: http
    nodePort: 30001
  selector:
    app: productpage
```

