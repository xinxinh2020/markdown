[TOC]



## 重点知识点

大写开头的名字是公有的

可以使用govendor来管理依赖：

```shell
go get -u github.com/kardianos/govendor
govendor init  # 配置好环境变量，在具体项目下执行
govendor get github.com/gorilla/mux #  下载mux到公共的vendor目录下
govendor add +external # 将依赖包加入到项目vendor目录
```



## 重要原理



### 为什么声明的语法与传统语言不同

https://blog.golang.org/declaration-syntax

c语言中，类型和声明的语法是一样的。

