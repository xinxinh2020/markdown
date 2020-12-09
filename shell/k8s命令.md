[TOC]

# K8S常用命令

```shell
kubectl version # 查看kubectl版本
kubectl cluster-info # 查看集群信息
kubectl api-versions # 获取当前 k8s 支持的 <group>/<version> 信息
kubectl api-resources # 获取当前 k8s 支持的资源列表 
											# --namespaced=true

kubectl rollout status deployments/kubernetes-bootcamp # 查看更新状态
kubectl rollout undo deployments/kubernetes-bootcamp # 回滚更新

kubectl create configmap test-config --from-file=configmap/ # 从文件中创建一个configmap
	--from-env-file # env-file中只包含形如：allowed="true"的键值对配置
kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm # 从字面量中创建一个configmap

# 创建对象
kubectl run # 创建一个Deployment 
kubectl expose # 创建一个Service 
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080 # 创建一个service,把内部的8080端口暴露出来，映射到Node端口
kubectl autoscale # 创建一个Autoscaler 
kubectl create <objecttype> [<subtype>] <instancename> # 创建对象，通用方法，需要用户知道objecttype
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 # 创建一个deployment

# 更新对象
kubectl scale # 水平伸缩一个控制器，如Deployment
kubectl scale deployments/kubernetes-bootcamp --replicas=4 # 将deployment的pod数量增加到4个副本 
kubectl autoscale deployment dep_name --cpu-percent=90 --min=1 --max=10 # Horizontal Pod Autoscaler
kubectl annotate # 给对象添加/减少注释
kubectl label # 给对象添加/减少一个标签
kubectl set <field> # 设置对象的一个属性
kubectl edit # 在编辑器中打开活动对象的配置，直接编辑其原始配置
kubectl patch # 

kubectl replace --force -f *.yaml # 重建一个对象
kubectl proxy # 运行一个代理，对外暴露HTTP API

kubectl get [deployments|nodes|pods|services|serviceaccounts|secret|apiservices|rs|statefulset|endpoints]# 列出所有指定类型的对象
	-l # 指定标签
kubectl -n kube-system get cm kubeadm-config -oyaml # 获取集群配置信息
kubectl describe [pod|deployment] # 查询对象详情
kubectl delete [deployment|node] # 删除对象
kubectl get [pods|services] -l run=kubernetes-bootcamp # 指定label获取pods
kubectl delete service kubernetes-bootcamp # 删除service
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2 # 修改image镜像
kubectl rollout status deployments/kubernetes-bootcamp # 查看版本更新进度
kubectl label pod $POD_NAME app=v1 # 给pod加上一个标签:app=v1
kubectl label nodes ln27 type=elastic # 给node打上一个标签
kubectl logs $POD_NAME # 应用打印到标准输出的信息可以用logs操作查到
kubectl exec $POD_NAME pwd # 直接在容器里执行命令，在只有一个容器的时候，可以省略容器的名字
kubectl exec -ti $POD_NAME bash # 进入到容器里面

kubectl config view # 查看$KUBECONFIG中包含的配置信息
kubectl config set-context ns_ctx --namespace=pangeo # 设置上下文
kubectl config use-context ns_ctx # 使用上下文

# kubeadm
kubeadm config images list # 查看需要哪些镜像
kubeadm reset # 重新安装时，在执行kubeadm init前需要reset一下
kubeadm join # 将一个node加入集群中
			# --control-plane : 作为master加入
kubeadm init phase upload-certs --upload-certs # 重新生成并上传证书
										   # 密码和解密密钥将在两小时后过期
				
kubectl get pod {podname} -n {namespace} -o yaml | kubectl replace --force -f - # 重启pod
kubectl cp 1.txt mysql-5fb6c74b86-xg97j:/1.txt # 拷贝文件

kubectl taint nodes ln15 node-role.kubernetes.io/master=true:NoSchedule # 禁止master部署pod

kubectl rollout history deployment coding # 查看修改历史
kubectl rollout undo deployment/abc # 回退到上个版本
kubectl rollout undo daemonset/abc --to-revision=3 # 回退到指定版本

kubectl convert -f xxx.yaml --output-version=apps/v1 # 将 GV 转换成 apps/v1
```

常用参数：

```shell
-n # 指定namespace
-l # 指定标签
--kubeconfig # 指定配置文件
```

