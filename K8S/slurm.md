





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

