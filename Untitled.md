[TOC]

## 开发效率和运行效率的平衡

说起Go语言，大家会不自觉地将其与C语言比较，普遍认为“Go=C+GC+Goroutine”，同时也将其称为“云计算时代的C语言”，在开发效率和运行效率之间取得了绝佳的平衡。Go语言既适应互联网应用的极速开发，又能在高并发、高性能的开发场景中如鱼得水。

还通过简单的生产者/消费者模型，通俗易懂地诠释了Go语言的并发编程哲学的口号：“Do not communicate by sharing memory；instead，share memory by communicating.”（不要通过共享内存来通信，而应通过通信来共享内存）

## 并发模型 CSP

## 值传递

for i 比 for...range 好用的时候
```
	for i := 0; i < len(ps); i++ {
		ps[i] = strings.TrimSpace(ps[i])
	}
	
	for i,_ := range ps {
	    ps[i] = strings.TrimSpace(ps[i])
	}
```
for...range  不能直接修改

## 并行编程
### Goroutine
尽管Goroutine 和 系统线程的区别实际上只是一个量的区别，但正是这个量变引发了Go语言并发编程质的飞跃：
1. 每个系统级线程都会有一个固定大小的栈（一般默认可能是2MB），这个栈主要用来保存函数递归调用时的参数和局部变量，固定了栈的大小导致了两个问题：一是对于很多只需要很小的栈空间的线程是一个巨大的浪费；二是对于少数需要巨大栈空间的线程又面临栈溢出的风险；一个Goroutine会以一个很小的栈启动（可能是2KB或4KB），当遇到深度递归导致当前栈空间不足时，Goroutine会根据需要动态地伸缩栈的大小（主流实现中栈的最大值可达到1GB）。因为启动的代价很小，所以我们可以轻易地启动成千上万个Goroutine
2. Go的运行时还包含了其自己的调度器，这个调度器使用了一些技术手段，可以在n个操作系统线程上多工调度m个Goroutine。Go调度器的工作原理和内核的调度是相似的，但是这个调度器只关注单独的Go程序中的Goroutine。Goroutine采用的是半抢占式的协作调度，只有在当前Goroutine发生阻塞时才会导致调度；同时发生在用户态，调度器会根据具体函数只保存必要的寄存器，切换的代价要比系统线程低得多。运行时有一个runtime.GOMAXPROCS变量，用于控制当前运行正常非阻塞Goroutine的系统线程数目

### 原子操作（同步）

1. 使用 `sync/atomic` 来保护数值型的共享资源
2. 互斥锁的代价比普通整数的原子读写高很多，在性能敏感的地方可以增加一个数字型的标志位，通过原子检测标志位状态降低互斥锁的使用次数来提高性能
3. 使用 `sync.Once` 来同步只需要初始化一次的变量
4. 使用 `atomic.Value` 来对基本数值类型及复杂对象的读写提供原子操作的支持

### 异步通信

对于从无缓存通道进行的接收，发生在对该通道进行的发送完成之前

### 并发模式

在并发编程中，对共享资源的正确访问需要精确地控制，在目前的绝大多数语言中，都是通过加锁等线程同步方案来解决这一困难问题，而Go语言却另辟蹊径，它将共享的值通过通道传递（实际上多个独立执行的线程很少主动共享资源）：为了提倡这种思考方式，Go语言将其并发编程哲学化为一句口号：“不要通过共享内存来通信，而应通过通信来共享内存。”（Do notcommunicate by sharing memory; instead, share memory bycommunicating.）

并发编程的核心概念是同步通信，但是同步的方式却有多种：

1. 互斥量sync.Mutex
2. 使用通道
3. 使用 sync.WaitGroup 来实现等待N个线程完成后再进行下一步的同步操
4. 生产者/消费者模型

## gRPC
## 函数式编程
## 闭包
闭包对捕获的外部变量并不是以传值方式访问，而是以引用方式访问。

## 引用与指针
for...each

## 接口
对于基础类型，是不支持隐式转换的，Go 对基础类型的类型一致性要求可谓是非常的严格，但对接口类型的转换则非常灵活

## 鸭子编程
If it walks like a duck and quacks like a duck, it must be a duck.
一个对象的特征不是由父类决定的，而是通过对象的方法决定。
我们不关心对象是什么类型，到底是不是鸭子，只关心行为./

## 错误处理

Go 语言库的习惯：一般不会产生 panic，即使在包内部使用了 panic, 在导出函数时也会被转化为明确的错误

painic 发生后，已经注册的 defer 仍然会执行

必须在 defer 函数中直接调用 recover()，不能对 recover() 进行包装，也不能在 defer 中嵌套 defer

## 其他

要使用 cgo 特性，编译时需要安装 C/C++构建工具链，在 macOS 和 Linux 下需要安装 GCC，在 Windows 下需要安装 MinGW，同时需要保证环境变量 CGO_ENABLED 为 1。在本地构建时，CGO 默认时启用的，在交叉构建时默认是禁用的

