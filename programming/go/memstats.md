



### 读取内存状态

```go
	var m runtime.MemStats
	runtime.ReadMemStats(&m)
```

MemStats 属性：

- Alloc：堆对象分配的内存字节数，包括了 reachable 的对象和还没被 gc 的 unreachable 对象
- HeapAlloc：和 Alloc 一样
- Mallocs：堆上创建的累积对象数量
- HeapObjects：堆上创建的对象数量（类似 Alloc）
- Frees：堆上释放的对象数量
- TotalAlloc：堆对象分配的内存的累积字节数，只增不减
- Sys：从 OS 获得的内存的总字节数，包含了堆，栈，和其他内部数据结构使用的内存。这些内存可能包含了 Swap 内存
- HeapSys：从 OS 获得的内存字节数，包括还没使用的堆内存
- HeapIdle：空闲但还没有(也可能已经)返回给 OS 的内存 span，调用 debug.FreeOSMemory() 可以强制把内存返回给 OS
- HeapReleased：返回给 OS 的物理内存字节数



## debug.FreeOSMemory()

 debug.FreeOSMemory() 可以强制把内存返回给 OS，但不是推荐的做法，应该让 go runtime 来处理这件事。