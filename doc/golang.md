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

### 错误处理

pkg/errors包,可以生成堆栈信息。注意不要wrap两次，这样会打印2个堆栈。

错误处理的原则：当处理不了错误时，应该把错误往上抛，在最上层wrap错误就可以，这样整个堆栈信息都在，而且不会重复打印。如果太早wrap，就会出现重复打印的问题。

星光的errors：info用于存放错误码对应的信息，msg用于存放原始错误信息。wrap的时候，只是附加一个错误码（以及错误码对应的信息info）。如果再次wrap的话，会覆盖掉上次的错误码及info，这是没有问题的。但使用New,Info等其他方法，相当于重新创建了一个新的error，这个error的堆栈就只能从当前堆栈开始，如果是在处理调用的方法返回的error，最好是用wrap把原来的error包起来，而不是新建一个error（有问题，把普通的error wrap起来，堆栈也只有到当前层？？？）。超时的话error会被吃掉

规范：

1. 如果不处理异常的话，要把原来的异常往上抛（把异常wrap一下再抛也可以），不要新建出新的异常再往上抛，这样会丢失现场
2. 如果要处理异常的话，最好先把异常堆栈打出来，保留下现场
3. 如果向上抛出的异常不是直接抛到框架层的，没有必要的话建议不要wrap，让上层丢到框架上的时候去warp，但是上层在wrap的时候，也要考虑要不要覆盖底层的错误码（建议提供一个接口，没有设置错误码就设置上去，设置了错误码就忽略）

### 为什么声明的语法与传统语言不同

https://blog.golang.org/declaration-syntax
