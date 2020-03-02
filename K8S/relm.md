[TOC]



## 部署安装

```shell
# 下载安装包
#https://get.helm.sh/helm-v3.1.0-linux-amd64.tar.gz
wget https://get.helm.sh/helm-v2.16.3-linux-amd64.tar.gz
tar zxvf helm-v2.16.3-linux-amd64.tar.gz
cd linux-amd64/
mv helm /usr/bin/
kubectl --namespace kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller --wait
# 有个tiller镜像可能下不了，要自己fq去下:gcr.io/kubernetes-helm/tiller:v2.16.3

```

## 常用命令

```shell
helm repo add stable https://kubernetes-charts.storage.googleapis.com/ #添加仓库
helm search repo stable # 列举1⃣️添加仓库中的chart
helm search hub wordpress # 列举所有公共可见的chart
helm install stable/mysql --generate-name # 安装mysql
helm install foo foo-0.1.1.tgz # 本地安装
helm install foo https://example.com/charts/foo-1.2.3.tgz # 在线安装
helm show chart stable/mysql # 显示chart的简介
helm show all stable/mysql # 显示chart的详情
helm status <release name> # 查看release的名字
helm delete <release name> # 删除release
helm show values stable/mysql # 显示该chart的可选配置项，可以把这些配置项保存在yaml文件中，修改yaml文件中的配置项，在安装的时候使用该yaml文件来做自定义配置
helm list # 列出已安装的release
```