

[TOC]

# 原理图

![aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMS8yOS8xNmViNTVhYTY2ODRiZmY0](/Users/xinxinhuang/workspaces/github/markdown/K8S/image/informer/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS8xMS8yOS8xNmViNTVhYTY2ODRiZmY0.jpeg)



创建informer的工厂方法：

```go
// NewSharedInformerFactoryWithOptions constructs a new instance of a SharedInformerFactory with additional options.
func NewSharedInformerFactoryWithOptions(client kubernetes.Interface, defaultResync time.Duration, options ...SharedInformerOption) SharedInformerFactory {
	factory := &sharedInformerFactory{
		client:           client,   // clientset
		namespace:        v1.NamespaceAll,  // 所有命名空间
		defaultResync:    defaultResync,  // 主动去list的周期
		informers:        make(map[reflect.Type]cache.SharedIndexInformer),  // 可以生成的informer列表
		startedInformers: make(map[reflect.Type]bool),   // 
		customResync:     make(map[reflect.Type]time.Duration),
	}

	// Apply all options
	for _, opt := range options {
		factory = opt(factory)
	}

	return factory
}
```



# 源码

## SharedInformer

```go
type SharedInformer interface {
    AddEventHandler(handler ResourceEventHandler)
    AddEventHandlerWithResyncPeriod(handler ResourceEventHandler, resyncPeriod time.Duration)
    GetStore() Store
    GetController() Controller
    Run(stopCh <-chan struct{})
    HasSynced() bool
    LastSyncResourceVersion() string
    SetWatchErrorHandler(handler WatchErrorHandler) error
}
```

SharedInformer提供了它的客户端到给定对象集合的权威状态的最终一致链接。 一个SharedInformer提供对一个特定API组和类型/资源对象的链接（如Deployment,StatefulSet）。一个SharedInformer的链接的对象集合在之后可能被限制为某个namespace，或者label selector，或者field selector。

对象的权威状态是apiservers提供的，对象将经历严格的状态序列。一个对象状态是(1)ResourceVersion和其他适当的内容，或者(2)“absent”

SharedInformer维护了一个本地的cache，通过 GetStore()获取，cache最终会和权威状态保持一致。这意味着，除非出现了持续性的通信问题，？？？？

对于一个给定的Informer和一个相关的对象ID X，informer的cache中出现的状态序列是一系列和X关联的连续的状态。一些状态可能不会出现在cache中，但出现的状态的顺序是正确的。但是需要注意的是，不同对象之间看到的状态是不保证有序的。

本地cache在开始时是空的，在Run()期间会被更新。

作为一个简单的例子，如果一个对象集合的状态不再改变，创建一个SharedInformer链接到该集合，SharedInformer Run()之后，它的cache会最终会持有这个集合精确的拷贝（除非它很快就停止了，或者。。。）

另外一个简单的例子，如果本地cache曾经持有一个对象的状态，并且这个对象最终被移除了，本地cache最终也会移除这个对象。

Store中的key的形式为：namespace/name（有命名空间的对象）或 name（没有命名空间的对象）,Clients可以使用`MetaNamespaceKeyFunc(obj)`从对象中获取它的key，也可以使用`SplitMetaNamespaceKey(key)`来把key切割开。

这里的client通过ResourceEventHandler来标识。 informer的本地cache每次更新以及client在run()之前加进来时，client会被告知这个更新。client通知在相应的cache更新和相应的index更新时.

对于一个给定的informer和client，通知会被循序地传递，对于一个给定的informer,client和对象id，通知会被有序地传递。

`ObjectMeta.UID`不会被用来区分对象。

一个client推荐处理每个通知，SharedInformer并不是设计来处理大量积压通知的。耗时较长的处理应该被放到别的地方，比如：`client-go/util/workqueue`

## ResourceEventHandler

```go
type ResourceEventHandler interface {
	OnAdd(obj interface{})
	OnUpdate(oldObj, newObj interface{})
	OnDelete(obj interface{})
}
```

