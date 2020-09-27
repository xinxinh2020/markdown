

# basic

```sh
go env # 查看go的环境变量
```



# module

```sh
go list -m all # 列举当前模块及其所有依赖
go mod init example.com/hello # 初始化模块
go mod tidy # 检测目录下所有引入的依赖，写入go.mod文件
go mod download # 下载依赖到GOPATH
go mod vendor # 将GOPAH下的依赖转移至该项目目录下的vendor目录
```

