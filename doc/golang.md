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

rune是int32的别名

unsafe.Sizeof()返回变量占用的字节数

string,数组,结构体都是值类型，不是引用类型，值类型的变量直接存储值，内存通常在栈空间分配

输入可以用fmt.Scanf()或fmt.Scanln()

switch的case后面不用带break,如果要让一个case匹配后继续匹配后面的case，可以使用fallthrough。default是必须的，所有case都不匹配后才会匹配到default。switch和case后面都可以接一个表达式，case还可以接多个表达式

switch x.(type)可以用来判断x的类型并根据不同类型做相应的处理

For-range方式遍历字符串可以自动识别中文

break和continue可以通过标签来跳出多层嵌套的循环

goto可以无条件跳到指定标签，但一般不推荐使用

Import的是包所在的文件夹路径，使用时的名字使用的是包名，即文件的package定义的名字

type myInt in可以给int取别名，取完别名后myInt和int就是两个不同的类型了

可以在函数声明中给返回值命名，这样return的时候就不用指明要return的变量了

每个源文件都可以包含一个init函数，init函数会被自动执行。执行顺序：变量定义->init->main

defer语句会被压入栈中，如果包含变量，变量的值也会被压入到栈中，等到函数执行完后才从栈中依次取出defer语句执行

函数外声明的变量都是全局变量

slice底层包括了起始地址，len和cap

通过make创建的切片，底层也有一个数组，该数组由make维护，对程序员不可见

结构体的所有字段在内存中是连续的

结构体之间进行类型转换时，要求两个结构体的字段要完全相同（包括字段名）

结构体实现String()方法的作用和Java的toString()类似，但在Println()中要传指针

对于方法而言，接受者为值类型或指针类型，都可以同时支持用值类型和指针类型的变量进行调用，实际使用的是值类型还是指针类型取决于方法声明，而不是调用

golang的结构体没有构造方法，可以用工厂模式实现类似构造方法的东西（也可以用单例模式）

结构体继承了其他结构体时，变量的访问采用就近访问，当父子结构体都定义了同一个字段时，或者多重继承的两个父类都有相同字段时，就需要显式指定使用哪个父类的字段

继承解决的主要问题是代码的复用性和可维护性，接口的价值在于设计好各种规范（方法）

golang中多态是通过接口来实现的,有两种形式：多态参数和多态数组

将接口转换成具体类的时候失败会报panic,可以使用类型断言用ok来接收类型转换是否成功，这样就不会报panic了

x.(type)可以得到x的实际类型

单元测试的名称需要以_test.go结尾，前缀无所谓，方法名需要是TestXxx()，Xxx也可以随意

协程的特点：有独立的栈空间；共享堆空间；由用户调度；可以看作轻量级的线程

golang可以很轻松开启上万个协程

通过runtime.NumCPU可以得到机器的逻辑cpu个数，通过GOMAXPROCS()可以设置使用的cpu个数

常量名并没有要求全大写的规范



### Printf的格式

```shell
%T # 打印一个变量的类型
%c # 打印字符
%v # 按照变量的值输出
%t # bool值
```



### 类型转换

基本数字类型之间的转换：直接用int32(),int64()等方法

基本数字类型转string：fmt.Sprintf()或者用strconv,如strconv.Itoa()

string转基本数据类型：strconv.ParseXXX()



### 管道

channel的本质是个队列，队列的容量是固定的，不能动态增长。管道是线程安全的，并且是有类型的。

可以用for range遍历管道，但遍历时，管道必须时已关闭的，否则会报错。更见的场景上使用select来遍历channel的数据

如果一个管道只有写没有读，运行时会报错。如果读写的频率不一样，不会有问题。

select用来监听和channel有关的io操作，如果没有default分支，select语句会一直阻塞，直到有一个case上的io操作可以进行



### 反射

#### Type

reflect.TypeOf()可以得到对象的类型（reflect.Type）

Type可以拿到类型的Kind(struct,int等，特别的，指针的类型为ptr),Name(类型名，如Student等)

#### Value

reflect.ValueOf()可以得到对象的值（reflect.Value）

Value可以拿到的对象各个字段的值，调用对象的方法等，调用对象的具体类型（Type）

Value的Int(),Float(),Bool()方法可以拿到对应类型的对象的值，但必须保证该对象的类型是正确的，否则会报panic

Value的Elem()可以返回指针指向的值对应的Value，拿到该Value之后再调用SetInt(),SetXxx()方法可以修改它的值，修改结果会影响到外面，即调用ValueOf传进来的指针指向的数据



### 字符串函数

```shell
len(str) # 字符串的字节数
[]rune(str) # 把字符串转成rune切片
[]byte(str) # string转byte切片
string([]byte{97,98,99}) # byte切片转string
strconv.Atoi("123") # 把string转成int
strconv.Itoa(123) # int转string
strconf.FormatInt(123,8) # 把十进制的int转成指定进制的string

# strings
strings.Contains("helloworld","hello") # 判断string中是否含有指定子串
strings.Count("hello","l") # 统计string中有几个子串
strings.EqualFold("abc","Abc") # 比较两个字符串是否相等，不区分大小写
strings.Index("helloworld","world") # 返回子串第一次出现的位置，没有则返回-1
strings.LastIndex("helloworld","world") # 返回子串最后一次出现的位置，没有则返回-1
strings.Replace("go go hello","go","java",n) # 字符串替换，n指定要替换几个，-1则全部替换
strings.Split("hello,world",",") # 将字符串按指定分隔符切割成数组
strings.ToLower("Go") # 转成小写
strings.ToUpper("Go") # 转成大写
strings.TrimSpace(" hello,world  ") #将字符串左右两边的空格去掉
strings.Trim("!hello! "," !") #将字符串左右两边指定的空格和!去掉
strings.TrimLeft("!hello! "," !") #将字符串左边指定的空格和!去掉
strings.TrimRight("!hello! "," !") #将字符串右边指定的空格和!去掉
strings.HasPrefix("http://www.baidu.com","http") # 判断字符串是否有xxx前缀
strings.HasSuffix("http://www.baidu.com","com") # 判断字符串是否有xxx后缀
```



### 日期相关函数

```shell
# import time
now := time.Now() # 返回当前时间，类型是time.Time
now.Year()
int(now.Month()) 
now.Day() # 返回年月日
now.Format("2006/01/02 15:04:05") # 格式化日期
#time.Duration是int64的别名
time.Nanosecond # 类型是Duration，值是1，表示一纳秒
time.Sleep(time.Second) # 休眠1s
time.Unix # 获取时间戳（从1970.1.1到当前时间的秒数）
time.UnixNano # 获取时间戳（纳秒）
```

### 内置函数

```shell
new(int) # 用来分配值类型的内存，包括结构体
recover() # 用于捕获panic，一般和defer配合使用
panic() # 抛出异常，如果没有被recover的话，程序就会终止
len() # 长度
cap()
append() # 给切片追加元素
copy(slice1,slice2) # 拷贝切片，把slice2拷贝给slice1
delete(map1,"key1") # 删除map中指定key的元素，如果key不存在，则不做任何操作，也不会报错
close() # 关闭管道，关闭后不能再写数据(会报错)，但可以读之前写入的数据，当之前写入的数据被取完后继续取的时候，不会报错，但是x,ok := <-chan中的ok会是false
```

### 文件操作

```shell
os.File # 封装了所有对文件的操作
```



### 排序

```shell
sort.Ints() # 给int切片按递增排序
sort.Sort(data Interface) #
```



## 重要原理

### 基础

golang里没有short,long,long long等整型，而是int8,int16,int32等，也没有double，而是float32,float64，应该是设计者希望能够让使用者更明确他正在使用的数据类型在内存中是几个字节的

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

