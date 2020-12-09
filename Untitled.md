针对插件端必传参数 (单独使用 nhctl 没有以下限制，因为用户还可以通过外挂 config 的方式传参：

##### 所有类型必传参数

-  -n, --namespace string   命名空间
- -t, --type string 应用的类型: helm or mainfest or helm-repo

##### manifest 和 helm 类型必传参数

- -u, --git-url string   项目的 git 地址
- --resource-path string  需要安装的 manifest 或 charts 资源相对于 git 根目录的路径，可以传空字符串，表示 manifest 资源就在根目录  

##### helm-repo 类型必传参数

- --helm-chart-name string   chart的名字
- --helm-repo-name/--helm-repo-url     二选一，repo的名字/repo的url

