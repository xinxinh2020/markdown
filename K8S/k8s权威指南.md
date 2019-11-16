

## 第一章

1. 每个Pod都有一个特殊的被称为根容器的Pause容器
2. Annotation里的是还没正式加入核心和扩展API资源对象的属性
3. Static Pod：没有被存放在etcd里，而是被存放在某个具体的Node上的一个具体的文件中，只能在这个Node上启动运行
4. Endpoint：Pod IP + containerPort
5. 申请CPU的最小单位一般为千分之一个
6. Label Selector可以类比为sql语句中的where查询条件，如name=redis-slave这个Label Selector作用于Pod时，可以被类比为select * from pod where name = 'redis-slave'。多个Selector表达式可以用,分隔，他们之间是AND的关系
7. k8s的Service其实就是我们经常提到的微服务架构中的一个微服务
8. 运行在每个Node上的kube-proxy进程其实就是一个智能的软件负载均衡器，负责把对Service的请求转发到后端的某个Pod实例上
9. Cluster IP仅仅作用于Service这个对象，无法Ping通，只能结合Service Port组成一个具体的端口，并且只能在集群内访问

