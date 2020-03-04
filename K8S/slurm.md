[TOC]

## 安装

```shell
# 创建munge key
yum install epel-release
yum install munge munge-libs munge-devel -y
yum install rng-tools -y
rngd -r /dev/urandom

/usr/sbin/create-munge-key -r
dd if=/dev/urandom bs=1 count=1024 > /etc/munge/munge.key
chown munge: /etc/munge/munge.key
chmod 400 /etc/munge/munge.key
chown -R munge: /etc/munge/ /var/log/munge/
chmod 0700 /etc/munge/ /var/log/munge/
systemctl enable munge
systemctl start munge

# 创建slurm rpm包
wget https://download.schedmd.com/slurm/slurm-20.02.0.tar.bz2
yum install rpm-build
yum install readline-devel perl pam-devel perl-ExtUtils-MakeMaker.noarch python3
yum install openssl openssl-devel pam-devel numactl numactl-devel hwloc hwloc-devel lua lua-devel readline-devel rrdtool-devel ncurses-devel man2html libibmad libibumad -y
yum install perl-ExtUtils-MakeMaker
yum install gcc
rpmbuild -ta slurm-20.02.0.tar.bz2

# 安装slurm rpm包
cd /root/rpmbuild/RPMS/x86_64
yum localinstall slurm-*.rpm
```



## 升级

```shell
yum install rpm-build
yum install readline-devel perl pam-devel perl-ExtUtils-MakeMaker.noarch
rpmbuild -ta slurm-19.05.3-2.tar.bz2 # 编译

# db/control节点
proxychains4 yum localupdate slurm-slurmd-19.05.3-2.el7.x86_64.rpm  slurm-19.05.3-2.el7.x86_64.rpm slurm-devel-19.05.3-2.el7.x86_64.rpm slurm-pam_slurm-19.05.3-2.el7.x86_64.rpm  slurm-slurmdbd-19.05.3-2.el7.x86_64.rpm  slurm-slurmctld-19.05.3-2.el7.x86_64.rpm   slurm-example-configs-19.05.3-2.el7.x86_64.rpm

# 非db节点
proxychains4 yum localupdate slurm-contribs-19.05.3-2.el7.x86_64.rpm slurm-19.05.3-2.el7.x86_64.rpm slurm-perlapi-19.05.3-2.el7.x86_64.rpm slurm-slurmdbd-19.05.3-2.el7.x86_64.rpm slurm-devel-19.05.3-2.el7.x86_64.rpm slurm-slurmd-19.05.3-2.el7.x86_64.rpm

```

