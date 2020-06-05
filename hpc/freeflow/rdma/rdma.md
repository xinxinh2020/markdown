[TOC]



## 查看主机上的RDMA网卡

```shell
[huangxinxin@mds0 ~]$ ip a
...
6: ib0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 2044 qdisc mq state UP group default qlen 256
    link/infiniband a0:00:02:20:fe:80:00:00:00:00:00:00:e4:1d:2d:03:00:50:d6:11 brd 00:ff:ff:ff:ff:12:40:1b:ff:ff:00:00:00:00:00:00:ff:ff:ff:ff
    inet 89.72.10.17/24 brd 89.72.10.255 scope global noprefixroute ib0
       valid_lft forever preferred_lft forever
    inet6 fe80::e61d:2d03:50:d611/64 scope link
       valid_lft forever preferred_lft forever
...
```





## Java程序使用RDMA

1. git clone https://github.com/zrlio/disni.git

2. 编译

   ```shell
   cd disni/
   
   # 编译源码
   mvn -DskipTests install
   
   # 编译libdisni
   cd libdisni
   # 安装依赖
   yum install libtool
   ./autoprepare.sh
   # Path为JDK的路径，eg:/root/jdk/jdk1.8.0_221
   ./configure --with-jdk=<path>
   make install
   # 成功安装后lib文件在/usr/local/lib目录下，ldconfig -n /usr/local/lib
   ```
```
   

3. 配置环境变量

   在/etc/profile中配置以下环境变量：

   ```shell
   export CLASSPATH=.:$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:/root/git/DiSNI/disni/target/disni-2.1-jar-with-dependencies.jar:/root/git/DiSNI/disni/target/disni-2.1-tests.jar
   export LD_LIBRARY_PATH=/usr/local/lib
```

   刷新环境变量，使其立即生效：

   ```shell
   source /etc/profile
   ```

   

4. 运行测试例子

   ```shell
   cd /root/git/DiSNI/disni/target/test-classes
   [root@vm01 test-classes]# cd /root/git/DiSNI/disni/target/test-classes
   [root@vm01 test-classes]# java com.ibm.disni.examples.ReadServer -a 127.0.0.1
   log4j:WARN No appenders could be found for logger (com.ibm.disni).
   log4j:WARN Please initialize the log4j system properly.
   log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
   Exception in thread "main" java.io.IOException: j2c::createEventChannel: rdma_create_event_channel failed: No such device
   
           at com.ibm.disni.verbs.impl.NativeDispatcher._createEventChannel(Native Method)
           at com.ibm.disni.verbs.impl.RdmaCmNat.createEventChannel(RdmaCmNat.java:60)
           at com.ibm.disni.verbs.RdmaEventChannel.createEventChannel(RdmaEventChannel.java:67)
           at com.ibm.disni.RdmaCmProcessor.<init>(RdmaCmProcessor.java:48)
           at com.ibm.disni.RdmaEndpointGroup.<init>(RdmaEndpointGroup.java:61)
           at com.ibm.disni.RdmaActiveEndpointGroup.<init>(RdmaActiveEndpointGroup.java:50)
           at com.ibm.disni.examples.ReadServer.run(ReadServer.java:50)
           at com.ibm.disni.examples.ReadServer.launch(ReadServer.java:105)
           at com.ibm.disni.examples.ReadServer.main(ReadServer.java:110)
   ```

   出现以上错误表明机器上没有支持RDMA的IB网卡设备。成功运行的结果类似下面：

   ```shell
   # 启动Server
   [root@power27 test-classes]# java com.ibm.disni.examples.ReadServer -a 89.72.10.27
   log4j:WARN No appenders could be found for logger (com.ibm.disni).
   log4j:WARN Please initialize the log4j system properly.
   log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
   ReadServer::server bound to address/89.72.10.27:1919
   
   # 在同一台机器上跑Client
   [root@power27 test-classes]# java com.ibm.disni.examples.ReadClient -a 89.72.10.27
   log4j:WARN No appenders could be found for logger (com.ibm.disni).
   log4j:WARN Please initialize the log4j system properly.
   log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
   ReadClient::initiated recv
   ReadClient::client connected, address /89.72.10.27:1919
   ReadClient::receiving rdma information, addr 140112235078592, length 100, key 66560
   ReadClient::preparing read operation...
   ReadClient::read memory from server: This
   ReadClient::read memory from server: This is a
   ReadClient::read memory from server: This is a RDMA/
   ReadClient::read memory from server: This is a RDMA/read
   ReadClient::read memory from server: This is a RDMA/read on st
   ReadClient::read memory from server: This is a RDMA/read on stag 66
   ReadClient::read memory from server: This is a RDMA/read on stag 66560 !
   ReadClient::read memory from server: This is a RDMA/read on stag 66560 !
   ReadClient::read memory from server: This is a RDMA/read on stag 66560 !
   ReadClient::read memory from server: This is a RDMA/read on stag 66560 !
   closing endpoint
   closing endpoint, done
   ```
   



