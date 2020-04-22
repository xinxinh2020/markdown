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
```



### 排序

```shell
sort.Ints() # 给int切片按递增排序
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

