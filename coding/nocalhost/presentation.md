

## helm 前置执行

``` shell
cd /data/tmp
git clone git@e.coding.net:codingcorp/bookinfo/bookinfo-charts.git
kubectl create ns demo10
helm upgrade --install -n demo10 --wait bookinfo-01 bookinfo-charts/ --debug
# 前置执行 done
# 依赖设置 todo
```

pre-hook 的定义：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-10"
  name: print-num-03
spec:
  template:
    spec:
      containers:
      - name: print-num
        image: print-num:01
        command: ["go", "run", "print-num.go", "30"]
      restartPolicy: Never
  backoffLimit: 4
```



- [ ] helm 依赖管理
- [ ] nocalhost 前置执行
- [ ] nocalhost 依赖管理
- [ ] 替换ruby开发容器

