## 安装OFED驱动

### 下载ISO驱动

下载rhel7.6的OFED驱动：MLNX_OFED_LINUX-4.6-1.0.1.1-rhel7.6-x86_64.iso

### 挂载ISO文件

```shell
  [root@power27 huangxinxin]# mount MLNX_OFED_LINUX-4.6-1.0.1.1-rhel7.6-x86_64.iso  mnt/
```

### 执行mlnxofedinstall进行安装

进入mnt目录，执行mlnxofedinstall进行安装

```shell
[root@power27 mnt]# ./mlnxofedinstall
Logs dir: /tmp/MLNX_OFED_LINUX.8548.logs
General log file: /tmp/MLNX_OFED_LINUX.8548.logs/general.log
```

 如果出现以下错误，则根据提示先安装相关依赖并加上--add-kernel-support参数进行安装

```shell
The 3.10.0-957.el7_lustre.x86_64 kernel is installed, MLNX_OFED_LINUX does not have drivers available for this kernel.
You can run mlnx_add_kernel_support.sh in order to to generate an MLNX_OFED_LINUX package with drivers for this kernel.
Or, you can provide '--add-kernel-support' flag to generate an MLNX_OFED_LINUX package and automatically start the installation.
  
[root@power27 mnt]# ./mlnxofedinstall --add-kernel-support
Note: This program will create MLNX_OFED_LINUX TGZ for rhel7.6 under /tmp/MLNX_OFED_LINUX-4.6-1.0.1.1-3.10.0-957.el7_lustre.x86_64 directory.
See log file /tmp/MLNX_OFED_LINUX-4.6-1.0.1.1-3.10.0-957.el7_lustre.x86_64/mlnx_iso.12298_logs/mlnx_ofed_iso.12298.log
Checking if all needed packages are installed...
ERROR: 'createrepo' is not installed!
'createrepo' package is needed for creating a repository from MLNX_OFED_LINUX RPMs.
Use '--skip-repo' flag if you are not going to set MLNX_OFED_LINUX as repository for
installation using yum/zypper tools.
/lib/modules/3.10.0-957.el7_lustre.x86_64/build/scripts is required to build mlnx-ofa_kernel RPM.
Please install the corresponding kernel-source or kernel-devel RPM.
Error: One or more required packages for installing OFED-internal are missing.
Please install the missing packages using your Linux distribution Package Management tool.
Run:
yum install kernel-devel-3.10.0-957.el7_lustre.x86_64 python-devel redhat-rpm-config rpm-build
Failed to build MLNX_OFED_LINUX for 3.10.0-957.el7_lustre.x86_64
  
# 安装依赖
[root@power27 mnt]# yum install python-devel redhat-rpm-config rpm-build createrepo gtk2 atk cairo gcc-gfortran tcsh
[root@power27 mnt]# yum install kernel-devel-3.10.0-957.el7.x86_64.rpm

# 正式安装，可能需要几分钟
[root@power27 mnt]# ./mlnxofedinstall --add-kernel-support
Note: This program will create MLNX_OFED_LINUX TGZ for rhel7.6 under /tmp/MLNX_OFED_LINUX-4.6-1.0.1.1-3.10.0-957.el7_lustre.x86_64 directory.
See log file /tmp/MLNX_OFED_LINUX-4.6-1.0.1.1-3.10.0-957.el7_lustre.x86_64/mlnx_iso.39640_logs/mlnx_ofed_iso.39640.log
Checking if all needed packages are installed...
Building MLNX_OFED_LINUX RPMS . Please wait...
Creating metadata-rpms for 3.10.0-957.el7_lustre.x86_64 ...
WARNING: If you are going to configure this package as a repository, then please note
WARNING: that it contains unsigned rpms, therefore, you need to disable the gpgcheck
WARNING: by setting 'gpgcheck=0' in the repository conf file.
Created /tmp/MLNX_OFED_LINUX-4.6-1.0.1.1-3.10.0-957.el7_lustre.x86_64/MLNX_OFED_LINUX-4.6-1.0.1.1-rhel7.6-ext.tgz
rpm --nosignature -e --allmatches --nodeps mlnx-ofa_kernel
rpm -e --allmatches --nodeps  kmod-mlnx-ofa_kernel
```

