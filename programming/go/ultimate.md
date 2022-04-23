

[TOC]

## 名言

### Legacy Software

- "There are two kinds of software projects: those that fail, and those that turn into legacy horrors" - Peter Weinberger(inventor of AWK)
- "Legacy software is an unappreciated but serious problem. Legacy code may be the downfall of our civilization" - Chuck Moore(inventor of Forth)
- "We think awful code is written by awful devs. But in reality, it's written by reasonable devs in awful circumstances" - Sarah Mei

### Metal Model

- One of our many problems with thinking is "connitive load": the number of things we can pay attention to at once. The cliche is 7+/- 2, but for many things it even less. We make progress by making those few things be more powerful. - Alan Kay
- The hardest bugs are those where your mental model of the situation is just wrong, so you can't see the problem at all - Brian Kernighan
- Debuggers don't remove bugs. They only show them in slow motion

### Performance

- The hope is that the progress in hardware will cure all software ills. Howerver, a critical observer may observe that software manages to outgrow hardware in size and sluggishness. Other observers had noted this for some time before, indeed the trend was becoming obvious as early as 1987 - Niklaus Wirth
- The most amazing achievement of the computer software industry is its continuing cancellation of the steady and staggering gains made by the computer hardware industry - Henry Petroski(2015)
- The hardware folks will not put more cores into their hardware if the software isn't going to use them, so, it is this balancing act of each other, and we are hoping that Go is going to break through on the software side - Rick Hudson(2015)

### Correctness

- Don't make coding decisions based on what you think might perform better. You must benchmark or profile to know if code is not fast enough. Then and only then should you optimize for performance.
- Make it correct, make it clear, make it concise, make it fast. In that order - Wes Dyer
- Good engineering is less about finding the "perfect" solution and more about understanding the tradeoffs and being able to explain them - JBD
- The basic ideas of good style, which are fundamental to write clearly and simply, are just as important now as they were 35 years ago. Simple, straightforward code is just plain easier to work with and less likely to have problems. As programs get bigger and more complicated, it's even more important to have clean, simple code - Brian Kernighan
- Problems can usually be solved with simple, mundane solutions. That means there's no glamorous work. You don't get to show off your amazing skills. You just build something that gets the job done and then move on. This approach may not earn you oohs and aahs, but it lets you get on with it - Jason Fried 

### Code Reviews

- The software business s one of the few palces we teach people to write before we teach them to read - Tom Love(inventor of Objective C)
- Making thiings easy to do is a false economy. Focus on making things easy to understand and the rest will follow - Peter Bourgon
- Everything should be made as simple as possible, but not simpler - Albert Einstein
- The purpose of abstraction is not to be vague, but to create a new semantic level in which one can be absolutely precise - Edsger W. Dijkstra
- Computing is all about abstractions. Those below yours are just details. Those above yours are limiting complicated crazy town - Joe Beda

### syntax

- Implicit conversion of types is a Hoalloween special of coding. Whoever thought of them deserves their own special hell - Martin Thompson



### other

- Polymorphism means that you write a certain program and it behaves differently depending on the data that it operates on - Tom Kurtz(inventor of BASIC)
- I think that the real problem with C is that it doesn't give you enough mechanisms for structruing really big programs, for creating "firewalls" within programs so you can keep the various pieces apart. It's not that you can't do all of these things, that you can't simulate object-oriented programming or other methodology you want in C. You can simulate it, but the compiler, the language itself isn't giving you any help 

Metal Mode: 心智模型，所写的代码要在脑海中有清晰的结构，能够快速找到想要找到的代码的位置。一个开发者的心智模型可能只装得下 1 万行代码。

优先使用心智模型和日志来找到问题所在，而不是 debugger。debugger 用来从头到尾跟进一个新项目的工作流很有好处。

性能问题主要来自以下 4 个方面：

- 网络 和  磁盘 IO 等延迟
- 内存分配 和 垃圾回收
- 访问数据的效率
- 算法的效率

一般用 var 来声明一个零值的变量，而不是使用短变量声明

隐式转换带来的坏处比好处多

匿名类型支持隐式转换，比如方法作为参数时就是使用匿名类型。常量（包括字面值常量）是支持隐式转换和类型提升的，Kind(字面值常量的类型 称为 Kind) 会被提升成 Type,低精度的会被提升为高精度的

内存分配时不够字长的部分会被填充，但不用特意考虑为了性能问题而去减少填充，它占不了多少内存

一个进程的栈大小是 1M，一个 Gorotine 的大小是 2K，如果 2K 不够用了，Go 会创建一个比原来大 25% 的栈，并把原来的栈拷贝过去，拷贝时，指向原栈的指针都会被更新

Go 的所有参数传递都是值传递，这种方式的好处是提供较好的完整性，但效率会比较差

逃逸分析：编译器会通过静态的分析出哪些内存是不能在栈上分配的，并把它们分配到堆上

尽量做到在栈上分配，因为栈上的内存是不需要进行垃圾回收了

给一个变量 s 赋值一个结构体时，尽量不要直接加上 & 将结构体的指针赋值给变量，但需要用到 s 的地址时再使用 &s，可以明确地知道 &s 取的是一个指针，提高可读性，也容易自己做逃逸分析

常量至少拥有 256 的精度

只有内置类型可以被声明为常量，常量只在编译期存在，可以享受隐式转换和类型提升等便利，这也是 Go 没有枚举类型的原因，因为枚举类型不是内置类型，所以无法享受这些便利

垃圾回收

- Gc  是并行的，每次运行的 gc stw 不会超过 100 微秒，总耗时会更多一些
- 为了并行收集，GC 最多占用 25% 的 CPU
- 可达性分析 + 标记清除？

如果声明了一个长度为 0 的切片（如 s := []string{}）而不是 nil 切片，这个切片的指针会指向一个全局唯一的一个 struct{},该 struct{} 也不占用内存

内存泄露的几大原因

- Goroutine 没有退出
- Map 中的 Key 没有及时删除

面向对象总是假设数据都是有状态和行为的，Go 大部分时候都会把状态和行为分开，所以函数大部分时候是第一选择

什么时候应该采用方法而不是函数呢，这个问题的本质是：什么时候数据应该有行为？ 

一般内置类型都应该使用值语义，除非是某些结构体的字段在序列化时需要有空的概念

引用类型一般也用值语义，除非是传递给函数做 decode 或 unmarshall 的时候

结构体如果不知道采用哪种语义，就优先使用指针语义，因为不是所有数据都能复制的。使用指针语义意味着对结构体对象的修改结果只是让该对象发生了一些改变，而不是一个新的对象，而值语义则是创建一个新对象，有点类似 string 或 Time

工厂方法返回的是值语义还是指针语义决定了这个结构体是使用哪种语义的

接口应该用来定义行为，而不是描述，不要定义 Animal,User 之类的接口

接口占用2个字，一个用来指向 iTable, 一个用来指向具体的数据（实现类型）

值语义的对象传给接口时，既可以使用值传递，也可以用指针传递，因为指针可以被转成值。但指针语义只能用指针传递，因为值不一定能取地址得到一个指针，比如当它是个常量的时候

使用值语义时，如果有必要可以同时使用指针语义（来做一些类似于 unmarshall 的操作），但使用指针语义时，就不要使用值语义了，因为我们不能假定指针指向的数据是可拷贝的 - 032

组合是用来嵌入行为（方法）的，不是用来嵌入状态（字段）的。使用 type 基于某种类型创建另一种类型时，如果不打算给新的类型添加行为，那这个类型一般没有创建的必要 - 037

优先使用函数，即使参数会很多，也不要尝试将参数封装到结构体然后用方法来简化它，这会让调用这个方法的人不知道那些字段是必须要初始化的，而作为函数参数就会明确要求调用者传过来

工厂方法一般返回具体类型，而不是接口

自定义错误类型最好不要导出，暴露相应的接口即可

错误处理一般包含了记录错误日志的步骤

和 Java 等不同，Java 的一个应用程序目录结构是在构建一个单体应用，但在 Go 中，每个目录都是一个静态库

包的层次关系只是给我们一个视图用来增加程序的结构化，但对于编译器来说，所有包都是平等的，没有子包的概念

作为工具包的 package 里一般不需要日志，比如标准库里就没有日志，如果真的需要写日志的地方，应该提供一个自定义函数让用户自己实现

Vendor 用来维护一份第三方依赖的拷贝

引入 internal 包时编译器会警告，internal 包里处于相同层级的包尽可能不要相互依赖

层次关系：cmd -> internal -> platform -> vendor

只有在 cmd 里允许 panic

线程从 CPU 核心上切换下来时需要保存大量状态，开销很大

Goroutine 在调用系统调用的时候，这个 goroutine 会被放到 network poller 里变成 wait 状态等待异步的系统调用，让出来的线程就可以执行其它 Goroutine

当 Goroutine 进行上下文切换时，线程并不需要进行切换，依然是 running 的状态，从而大大提高了线程的利用率,因此 Go 只需要几个线程，起太多线程很多时候并不能让程序跑得更快，反而会给操作系统的调度添堵

Go 把 I/O bound work 转换成了 CPU bound work

全局变量会被加载到每个 CPU 核到缓存中，并发读写时缓存会不断失效导致性能很差，所以在并发处理时应该谨慎使用全局变量

如果一个结构体里面有 mutex 字段，这个结构体就是不能复制的，因为复制会导致创建出一个新的 mutex

RWMutex 允许多个读同时进行，但只允许一个写，并且写的时候不允许读

go build/test --race 可以用来检测 Data Race

close 一个 channel 时可以让 for range 它的循环退出

带缓冲带 channel 大小要可预知，一般一个 Goroutine 分配一个，在往 channel 里发送信号的时候，要思考一个问题，如果 buff 满了会怎样

context 里只能存放通用数据（interfaces)

代码测试的覆盖率最好要达到 70~80%

leaf function (不会调用其它函数的函数) 如果代码比较少的话，可能会被编译器内联

## Code Reviews

四个主要的方面：Integrity, Readability, Simplicity, Performance

### Integrity

Two driving forces behind integrity:

- Integrity is about every allocation, read and wirte of memory being accurate, consistent and efficient. The type system is critical to making sure we have this micro level of integrity
- Integrity is about every data transformation being accurate, consistent and efficient. Writing less code and error handling is critical to making sure we have this macro level of integrity

Write less code: The industry average is around 15 to 50 bugs per 1000 lines of code

### Readability

代码库要让团队里处于平均水平的开发人员看得懂。低于平均水平的人要帮助他们赶上了，高于平均水平的人要让他们写的代码要让团队能看懂

### Simplicity

Simplicity is not hiding cost (like constructor), but complexity.

好的封装(方法)除了提供更好的可测试性，还应该提供更好的语义

## Design Philosophy 

### interfaces

- Interfaces give programs structure
- Interfaces encourage design by composition
- Interfaces enable and enforce clean divisions between components ???
- Decoupling means reducing the dependencies between components and the types they use
- Interfaces allow you to group concrete types by what they do
  - Don't group types by a common DNA but by a common behavior
  - Everyone can work together when we focus on what we do and not who we are
- Interfaces help your code decouple itself from change
  - you must do your best to understand what could change and use interfaces to decouple
  - Uncertainty about change is not a license to guess but a directive to STOP and learn more
- A good API is not just easy to use but also hard to misuse - JBD
- You can always embed, but you cannot decompose big interfaces once they are out there. Keep interfaces small - JBD

### package

- To be purposeful, packages must provide, not contain
  - Packages must be named with the intent to describe what it provides
  - Packages must not become a dumping group of disparate concerns (不要创建类似 model 的包，仅仅只是在里面放一堆结构体，即使在不同的包里定义了相同的结构体，也比这种情况好)
- To be usable, packages must be designed with reusability in mind

## 重点

### Lesson 3: Data Structures

#### 3.1 Data-Oriented Design

- If you don't understand the data, you don't understand the problem
- All problems are unique and specific to the data you are working with
- Data transformations are at the heart of solving problems. Each function, method amd work-flow must foucs on implementing the  specific data transformations required to solve the problems
- If your data is changing, your problems are changing. When your problems are changing, the data transformations needs to change with it
- Uncertainty about the data is not a license to guess but a directive to STOP and learn more
- Solving problems you don't have, creates more problems you now do
- Efficiency is obtained through algorithms but performance is obtained through data structrues are layouts

#### 3.2 Arrays

##### Mechanical Sympathy

- A cycle can run 4 instructions
- Latency of accessing L1,L2,L3,Main Memory: 4/11/39/107
- What we should prefer to happen is than cache line is already in L1 or L2 before we need it. What we have to do is write code  that creates predictable access patterns to memory if we want to be mechanically sympathetic with the hardware.
- If performance matters, then what we have to do is be much more efficient with how data gets in to the processor, not get the processors to run at higher clock speed.
- Predictable access patterns to memory is everything.
- If you allocate a contiguous block of memory, and walk through that memory on a predictable stride, the prefetchers,which is little software sitting inside the processor, can pick up on that data access and start bringing in those cache lines way ahead of when you need them.
- The cleanest and easiest way to create this predictable access pattern is to use an array. Prefetchers love arrays. The hardware loves arrays.
- Performance today is not about how fast we can churn that clock, how fast we can push the clock in terms of cycles per nanosecond, it's about how efficiently we can get data into the processor before we need it.



## 常用命令

```go
go test -cover // 代码测试覆盖率
go test -coverprofile [file_name] // 代码测试覆盖率报告
go tool cover -html [file_name] // 使用浏览器打开报告
go test -run none -bench . -benchtime 3s -benchmam -memprofile m.out // 不允许单元测试，运行所有 benchmark，测试内存分配情况
gocore -o core.txt PID // Write a dump of the running program
```



### Go 语言规范

如果声明变量的时候不做初始化，使用 var 而不要用 :=

用结构体给一个变量做初始化时，不要加 & 





什么时候使用值语义，什么时候使用指针语义？

