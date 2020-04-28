[TOC]





# Dockerfile命令

```shell
CMD ["jupyter" "notebook" "--ip" "0.0.0.0"] 
ENTRYPOINT ["/usr/local/bin/repo2docker-entrypoint"]
```



# 网络

## Infiniband

```shell
systemctl restart openibd # 重启IB驱动
```







## Linux

```shell
timedatectl # centos7 查看服务器时间
timedatectl set-timezone Asia/Shanghai # centos7 修改时区
sed -i "s/查找字段/替换字段/g"
python -mjson.tool # 将json字符串格式化显示

```

