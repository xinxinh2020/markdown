



## Daemon

每个用户会创建一个daemon，用于管理session。所有的mutagen session管理命令都会转发它们的操作到daemon，并简单地打印结果。



### Lifecycle

每个用户最多只会创建一个daemon，daemon的生命周期有3种管理方式：自动，手动，和系统。

#### 自动管理

如果daemon没有正在运行，会在执行session管理命令时自动启动，在系统关闭时自动退出。

#### 手动管理

通过`mutagen daemon start`和`mutagen daemon stop`管理

#### 系统管理

macOS和Windows可以通过`mutagen daemon register`让daemon开机自启动



## 传输协议

### SSH

mutagen需要先通过scp把一个agent的二进制拷贝到远程系统，再通过ssh命令来运行agent并和agent进行通信