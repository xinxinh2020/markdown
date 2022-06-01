## Cli 简化

### dev start

完整命令：nhctl dev start bookinfo -d authors -t deployment  --without-terminal  -s /Users/xinxinhuang/Workplace/github/bookinfo/bookinfo-authors --container authors  -i nocalhost-docker.pkg.coding.net/nocalhost/dev-images/golang:latest  -n bookinfo --kubeconfig /Users/xinxinhuang/.nh/vscode-plugin/kubeConfigs/15d4599c77cb77ae44f2bfa77e999829 

简化命令：nhctl dev start

需要配置的参数：

```yaml
workloadName: authors # required
container: authors # required
workloadType: deployment # optional, default: deploymentappName:  bookinfo # optional, default: default
namespace: bookinfo # optional, default: default
kubeconfig: ~/.kube/config # optional, defalut: ~/.kube/config
withoutTerminal: true # optional, default: true
devImage: nocalhost-docker.pkg.coding.net/nocalhost/dev-images/golang:latest # optional, default: image defined in devConfig
devMode: replace # replace/duplicate/mesh default: replace
```

### dev end

完整命令：nhctl dev end bookinfo -d authors -t deployment -n bookinfo --kubeconfig /Users/xinxinhuang/.nh/vscode-plugin/kubeConfigs/15d4599c77cb77ae44f2bfa77e999829 

简化命令：nhctl dev end

需要配置的参数：复用 dev start 的配置即可



## UI

写一篇公众号介绍一下怎么使用，以及如何参与开发



### 需要删减的功能

- 应用升级？
- 应用部署？