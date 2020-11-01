



参数：

- --create-namespace: 如果namespace不存在的话则创建相应的namespace，默认为false
- --dry-run: 模拟安装
- --no-hooks: 禁用hook执行，默认false
- --timeout: 每个单独的kubernetes操作的等待超时时间，如hook中定义的Job
- --wait: 等待所有的Pod，PVC，Service和Deployment/StatefunSet/ReplicaSet的最少量的Pod Ready
- --generate-name, -g: 生成一个名字，忽略NAME参数
- --description: 添加一个描述
- --dependency-update: 在安装chart之前执行helm依赖更新
- --atomic: 原子性，安装进程在失败的时候会删除本次安装的所有资源，执行此flag会自动添加--wait
- --replace: 使用并替换掉已经删除的 release 的名字

Values:

- --values,-f: 指定用于设置values的yaml文件，可以有多个
- --set: 设置values，以key1=val1,key2=val2的形式
- --version: 指定chart版本，默认为最新版本
- --verify: 在使用helm包前先验证一下它,默认false
- --repo: chart repository的url
- --useruname: chart 仓库的 username
- --password: chart 仓库的 password
- 

