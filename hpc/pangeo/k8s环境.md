[TOC]



```shell
# 配置阿里源
cat <<EOF > /etc/yum.repos.d/kubenetes_ali.repo
[kubernetes]
name=Kubernetes Repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
enabled=1
EOF

kubeadm reset
yum remove kubelet kubeadm kubectl
yum install -y kubelet-1.15.0 kubeadm-1.15.0 kubectl-1.15.0
systemctl enable kubelet
# kubeadm config images list --kubernetes-version

# master
kubeadm init --apiserver-advertise-address=172.26.253.135 --pod-network-cidr=10.244.0.0/16 --upload-certs --kubernetes-version v1.15.0 --image-repository registry.aliyuncs.com/google_containers
	# --apiserver-advertise-address指定要绑定的网卡，不指定则使用默认路由对应的IP地址
	# --control-plane-endpoint指定高可用模式下使用的负载均衡IP或DNS
	# --kubernetes-version指定版本，默认会使用最新版本
	# --pod-network-cidr
	# --upload-certs 主控制面板的证书会被加密并上传到Secret kubeadm-certs上
	# --kubernetes-version

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml

kubectl taint node  node-role.kubernetes.io/master-

# worker
kubeadm join 172.26.253.135:6443 --token 2mz6fj.rxt089m527glfa3j \
    --discovery-token-ca-cert-hash sha256:12cc98a74912ff6e6e2b66f02a2faedd19ab45ab689d57483ea35bc962108c5b
    
kubeadm join 47.92.250.196:6443 --token 2mz6fj.rxt089m527glfa3j \
    --discovery-token-ca-cert-hash sha256:12cc98a74912ff6e6e2b66f02a2faedd19ab45ab689d57483ea35bc962108c5b
```

