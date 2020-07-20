[TOC]



## 搭建

centos7

```shell
yum install -y curl policycoreutils-python openssh-server
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-12.10.9-ce.0.el7.x86_64.rpm
yum localinstall gitlab-ce-12.10.9-ce.0.el7.x86_64.rpm
```



修改配置 /etc/gitlab/gitlab.rb：

```shell
external_url 'http://localhost:8081'
unicorn['port'] = 9000
```

注意端口不要冲突了。

设置环境变量：

```shell
export LC_ALL=en_US.UTF-8 
export LANG=en_US.UTF-8 
export LANGUAGE=en_US.UTF-8
source ~/.bashrc
```

内存要大于2G。

启动：

```shell
gitlab-ctl reconfigure
gitlab-ctl restart
```



遇到的问题：

启动后访问界面出现502错误，可以尝试执行一下`gitlab-ctl hup unicorn`

## 常用命令

```shell
gitlab-ctl status # 查看各个服务的状态
```

