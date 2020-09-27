

[TOC]



### 准备工作

```shell
timedatectl set-timezone Asia/Shanghai
hostnamectl set-hostname k8s-01
# 禁用selinux
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

systemctl stop firewalld
systemctl disable firewalld

# 安装docker 
yum install yum-utils device-mapper-persistent-data lvm2.
yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo
yum update && yum install docker-ce-18.06.2.ce
mkdir /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
mkdir -p /etc/systemd/system/docker.service.d


systemctl restart docker
systemctl enable docker

# 关闭Swap
swapoff -a # 临时关闭
sed -i '/ swap / s/^/#/' /etc/fstab # 注释/etc/fstab包含swap那行，永久关闭


```

####  配置DNS服务

```shell
# 配置DNS
yum install -y dnsmasq
# 在/etc/dnsmasq.conf中添加以下配置:
ddn-hosts=/etc/dnsmasq.hosts
listen-address=127.0.0.1,192.168.1.123 # 192.168.1.123为主机IP

# 添加要解析的域名
vim /etc/dnsmasq.hosts

# 重启服务
systemctl restart dnsmasq
systemctl enable dnsmasq
```



#### 搭建HAProxy

```shell
yum install haproxy
# 配置/etc/haproxy/haproxy.cfg
systemctl enable haproxy
systemctl start haproxy
```



### 安装k8s

添加k8s的阿里源：

```shell
cat <<EOF > /etc/yum.repos.d/kubenetes_ali.repo
[kubernetes]
name=Kubernetes Repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
enabled=1
EOF
```



安装kubelet kubeadm kubectl ：

```shell
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

The kubelet is now restarting every few seconds, as it waits in a crashloop for kubeadm to tell it what to do.

配置cgroup:

```shell
docker info | grep -i cgroup 

# 如果cgroup不是cgroupfs的话(比如systemd)，则需要在/etc/sysconfig/kubelet中添加一下内容
KUBELET_EXTRA_ARGS=--cgroup-driver=<value>

# then restart it
systemctl daemon-reload
systemctl restart kubelet
```



查看依赖的容器镜像：

```shell
[root@vm01 ~]# kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.16.2
k8s.gcr.io/kube-controller-manager:v1.16.2
k8s.gcr.io/kube-scheduler:v1.16.2
k8s.gcr.io/kube-proxy:v1.16.2
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.15-0
k8s.gcr.io/coredns:1.6.2

# 无法访问k8s.gcr.io，从别的地方中转一下
docker pull aiotceo/kube-apiserver:v1.16.2
docker pull aiotceo/kube-controller-manager:v1.16.2
docker pull aiotceo/kube-scheduler:v1.16.2
docker pull aiotceo/kube-proxy:v1.16.2
docker pull aiotceo/pause:3.1
docker pull trustxz/etcd:3.3.15-0
docker pull aiotceo/coredns:1.6.2

# 改标签
docker tag aiotceo/kube-apiserver:v1.16.2 k8s.gcr.io/kube-apiserver:v1.16.2
docker tag aiotceo/kube-controller-manager:v1.16.2 k8s.gcr.io/kube-controller-manager:v1.16.2
docker tag aiotceo/kube-scheduler:v1.16.2 k8s.gcr.io/kube-scheduler:v1.16.2
docker tag aiotceo/kube-proxy:v1.16.2 k8s.gcr.io/kube-proxy:v1.16.2
docker tag aiotceo/pause:3.1 k8s.gcr.io/pause:3.1
docker tag trustxz/etcd:3.3.15-0 k8s.gcr.io/etcd:3.3.15-0
docker tag aiotceo/coredns:1.6.2 k8s.gcr.io/coredns:1.6.2

```



initialize the control-plane node:

```shell
# 第一个master
kubeadm init --apiserver-advertise-address=192.168.23.133 --control-plane-endpoint "k8s-haproxy:80" --pod-network-cidr=10.244.0.0/16 --upload-certs
	# --apiserver-advertise-address指定要绑定的网卡，不指定则使用默认路由对应的IP地址
	# --control-plane-endpoint指定高可用模式下使用的负载均衡IP或DNS
	# --kubernetes-version指定版本，默认会使用最新版本
	# --pod-network-cidr
	# --upload-certs 主控制面板的证书会被加密并上传到Secret kubeadm-certs上


export KUBECONFIG=/etc/kubernetes/admin.conf # root用户

#worker
kubeadm join k8s-haproxy:80 --token 8aqcvz.h4dvoqqcxbd8lez0 \
    --discovery-token-ca-cert-hash sha256:01dd77f4db405a7c4b43449fd20d1fae1f3dc66fb03bc946fe3587c0c5a537d1

#master
kubeadm join k8s-haproxy:80 --token 8aqcvz.h4dvoqqcxbd8lez0 \
    --discovery-token-ca-cert-hash sha256:01dd77f4db405a7c4b43449fd20d1fae1f3dc66fb03bc946fe3587c0c5a537d1 \
    --control-plane --certificate-key d3d2f1984a544a372f654725059f3bace1190a51c036d1df4e34d14ade7caa11
```

安装一个Pod网络：

The network must be deployed before any applications. Also, CoreDNS will not start up before a network is installed.

```shell
# 可选的网络类型有很多，这里用flannel
sysctl net.bridge.bridge-nf-call-iptables=1
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

kubectl get pods --all-namespaces # 查看所有pod是否都已正常启动
```

允许Pod创建在该控制面板节点上：

```shell
kubectl taint nodes --all node-role.kubernetes.io/master-
```

创建一个key：

```shell
kubeadm alpha certs certificate-key
# c71cd915a2c667f18c4c5be0f96869ced7f75775b7a631e92b51f89e7a30c763
```



添加其他节点到master节点：

```shell
# --control-plane表示作为控制面板节点添加
# --certificate-key 会让控制平面证书从kubeadm-certs下载，并使用该key对集群进行加密
kubeadm join k8s-haproxy:80 --token 8aqcvz.h4dvoqqcxbd8lez0 \
    --discovery-token-ca-cert-hash sha256:01dd77f4db405a7c4b43449fd20d1fae1f3dc66fb03bc946fe3587c0c5a537d1 \
    --control-plane --certificate-key d3d2f1984a544a372f654725059f3bace1190a51c036d1df4e34d14ade7caa11

# 注意DNS域名要配好
```



开启代理：

```shell
# 开启访问后外面的机器可以通过8001端口访问K8S Api接口
kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$'
```

### 安装metrics-server

```shell
git clone https://github.com/kubernetes-incubator/metrics-server.git
cd metrics-server/
docker pull mirrorgooglecontainers/metrics-server-amd64:v0.3.6
docker tag mirrorgooglecontainers/metrics-server-amd64:v0.3.6 k8s.gcr.io/metrics-server-amd64:v0.3.6
# 修改deploy/1.8+/metrics-server-deployment.yaml的imagePullPolicy为IfNotPresent
kubectl create -f deploy/1.8+/  # for Kubernetes > 1.8
```






### 安装Dashboard

```shell
docker pull siriuszg/kubernetes-dashboard-amd64:v1.10.1
docker tag siriuszg/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

kubectl describe serviceaccount kubernetes-dashboard -n kube-system # 找到其对应的secret
kubectl describe secret kubernetes-dashboard-token-hc7x2 -n kube-system # 找到token

```



### 验证

```shell
wget https://github.com/vmware-tanzu/sonobuoy/releases/download/v0.16.2/sonobuoy_0.16.2_linux_amd64.tar.gz
tar zxvf sonobuoy_0.16.2_linux_amd64.tar.gz

./sonobuoy run --wait
results=$(./sonobuoy retrieve)
./sonobuoy results $results
./sonobuoy status

./sonobuoy delete --wait
```

