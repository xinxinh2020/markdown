

[TOC]

# 基础

诞生于1995年，一开始就是为了做前端检验，比如用户名和密码的长度

Chrome的js引擎v8是目前最快的

标识符可以含有字母，数字，_和$

数据类型：String,Number,Boolean,Null,Undefined

整数和浮点数都是Number类型

Boolean只有两个值：true,false

Null只有一个值：null,用于表示一个空对象，用typeof null会返回object

Undefine只有一个值：undefined，用typeof检测到的类型是undefined，表示一个变量声明后未赋值.读取对象中没有定义的值不会报错，只会报undefined

可以使用typeof xxx检查变量类型,返回一个字符串

js进行浮点运算时可能得到不精确的结果

任何数据类型和字符串做拼接都会被转成字符串

在js中使用unicode编码：\u，后面的数字为16进制数字，在html中使用unicode编码：&#+十进制数字

==/!=会做自动类型转换，===/!==不会

{}包起来的代码，要么全执行，要么都不执行

prompt()可以接收用户输入 

可以创建标签来让for循环退出多重循环

调用函数时，不会检查实参的类型和数量，多传的参数不会被处理，少传的参数为undefined，不写return语句会返回一个undefined

函数外定义的变量都是全局变量，保存在全局对象window中，在js中，方法和函数没什么区别，函数其实就是window中的方法

使用var声明的变量，会在所有代码执行前声明（但不会被赋值），这个特性在全局作用域和函数作用域中都存在

函数声明（包括了函数体）会被放到代码最前，用匿名函数赋值给变量的方式不会被提前

js也有自动垃圾回收机制

字符串底层是以字符数组的形式保存的

js代码一般写在下面，以免加载的时候，js执行完了相关html元素还没加载进来，或者写在window.onload里。

使用没有定义的变量会报错，但是使用没有定义的属性（对象里的属性）不会，它的值是undefined

ajax有同源策略，来自一个网站的ajax默认只能访问该网站，访问其它网站需要其它网站允许跨域访问

websocket用于长连接（ajax是短连接），协议是ws，websocket不存在跨域问题

异步的实现方式：

- 回调：但会引入多层嵌套(回调地狱)
- Promise+Async



# 事件

- click：点击事件
- load：页面加载完



# 面向对象

## 常识

属性名无需遵守标识符的规范，但建议还是遵守一下比较好

obj1 == obj2 如果两个对象的地址不同，则返回false 

方法都有一个隐含的参数this，指向调用它的对象，需要注意的是，函数也是一个特殊的方法，它的this指向window

可以用 ` for attr in obj ` 枚举出对象的属性名，也可以用in来检测一个对象中是否含有相应的属性

使用in来判断一个对象中是否包含有该属性时，会去判断原型中的属性，可以使用hasOwnProperty()来检测该对象自己包含的属性（不包括原型中的）

let声明一个局部变量，var声明了一个全局变量。

## 构造函数

构造函数的定义和普通方法一样，但一般首字母为大写，使用的时候要用new关键字才能得到一个对象，否则就是调用普通函数

构造函数中的this引用的是新创建的对象，使用this来为对象增加属性：

```javascript
// Cat相当于是一个类
function Cat(){
  this.age = 10
  this.name = 'miao'
}

var cat = new Cat()
cat instanceof Cat // 检查cat对象是不是Cat的实例
```



## 方法/函数

函数其实也是一种特殊的方法，它的this指向的是window

每个方法（函数）都有一个prototype属性

通过构造函数创建的对象，都有一个prototype属性，指向同一个prototype对象，访问prototype对象方式：

```javascript
function Cat(){
  this.age = 10
  this.name = 'miao'
}

// 通过函数名
var p = Cat.prototype

var cat = new Cat()
//通过对象名
cat.__proto__
```

在prototype里放变量，可以实现类似类变量的效果，当通过对象访问一个属性时，会先从成员变量中找，没有再去类变量中找，如果本类变量中没有，还会去父类的类变量（原型）中找，直到找到Object的prototype。把成员方法放到prototype中定义，就不用每个对象都创建一个函数对象，可以节省内存

放在prototype里的变量才算真正属于这个类的成员，在对象中添加的属性，都只是该对象拥有而已

函数对象内置call和apply方法可以传一个对象进去，改变函数的this对象。call的参数在第一个参数（第一个参数是传给this的对象）之后依次传过去即可，apply的参数要放到数组中传给第二个参数

函数调用时有两个隐含的参数，一个是this，一个是arguments



## 内建对象

### Array

Array的索引也可以不是数字

length属性竟然是可以修改的（它是怎么做到修改一个属性的时候，同时把数组给扩容了？）

可以用字面量来创建：var arr = [1,2,3

数组元素的类型可以是任意类型

方法：

```javascript
// 可以查看api文档
push() // 向数组末尾添加元素
pop() // 删除末尾元素
unshift() //向数组开头添加元素
shift() //删除数组第一个元素
forEach(callBack) // callBack的类型为function(value,index,arr) 其中第三个参数为数组本身 ie8以上支持
slice(start,end)  // 返回一个切片
splice(start,end) // 删除指定切片
concat() // 连接多个数组
sort() // 默认按unicode编码排序，可以传一个callback(a,b)，根据返回值是true or false来决定
```



### Date

```js
getDate()  // 一个月中的日
getDay()  // 一周中的日
getTime()  // 时间戳 单位是毫秒
```



### Math

它不是一个构造函数，不需要创建对象，里面的方法直接调用就可以：

```js
Math.PI
Math.abs()
Math.sin()
Math.random()  // 生成0到1之间的随机数
Math.Max()
Math.pow() // 幂
Math.sqrt() // 平方
```

### String

```javascript
slice()
substring() //和slice类似
substr()  //没有包含在规范里，不推荐使用
split() // 字符串分割，可以传递一个正则表达式
search()  // 搜索字符串是否含有指定值，可以传递一个正则表达式，返回第一次出现的索引，没有则返回-1
match(/[a-z]/ig)  //  提取匹配的内容，可以使用正则表达式, i表示忽略大小写。g表示全局匹配，返回一个数组
replace()  // 替换字符串，可以使用正则表达式
toLowerCase()
```

### RegExp

```javascript
var reg = new RegExp("a") // 匹配包含有a的字符
var str = 'a'
var ok = reg.test(str)  // 测试字符串是否匹配

// 忽略大小写
var reg = new RegExp('a','i')
// 使用字面量创建
var reg = /a/i

// 常用正则
var reg = /a|b|c/ // 或 等价于 [abc]
var reg = /[a-z]/ // 小写字母
var reg = /[A-z]/ // 任意字符
var reg = /[^ab]/ // 不能只包含a或b
var reg = /ab{4}/ // 匹配abbbb
var reg = /(ab){4}/ // 匹配abababab
var reg = /ab{1,4}c/ // 中间的b出现1-4次都可以
var reg = /ab{2,}c/ // b出现2次及以上
var reg = /ab+c/ // b出现至少1次及以上
var reg = /ab*c/ // b出现0个及以上
var reg = /ab?c/ // b出现0个或1个
var reg = /^a/ //  以a开头
var reg = /a$/ //  以a结尾

// 元字符
. : 表示任意字符
\w : [0-9A-z_]
\w : [^0-9A-z_]
\d : [0-9]
\D : [^0-9]
\s : 空格
\S : 除了空格

//总结
[]表示或
{}表示重复出现次数
()表示组合（里面的字符串是一个整体）
^表示开头，如果在[]里则表示非
$表示结尾
```



## 包装类

将基本数据类型转换成包装类

```js
//Number
var num = new Number(3)
//String
var str = new String('hello')
//Boolean
var boo = new Boolean(true)
```

基本数据类型是没有方法和属性的，之所以还能够使用方法和属性，是因为在调用的时候，解释器自动把基本数据类型转成了对象



## DOM对象



### document

文档对象document放在全局对象window里，可以直接使用。

整个网页可以看作是一个DOM对象树，根对象是document，它在全局作用里，可以直接使用

DOM对象树里的其他元素对象可以通过getElementById()等方法获取：

```javascript
var btn = document.getElementById("btn")  // 获取一个元素
var btns = document.getElementsByTagName("button")  // 获取一组元素,*可以获取所有元素
var btn = document.getElementsByName("age")  // 获取一组元素
var eles = document.getElementsByClassName("") // ie8以上支持
var ele = document.querySelector('.box1 div') // 根据css选择器选元素，只返回一个
var ele = document.querySelectorAll('.box1') // 根据css选择器选元素，可以返回多个
```

dom增删改：

```javascript
var li = createElement('li') // 创建一个li元素
var txt = createTextNode('guangzhou')  // 创建一个文本节点
li.appendChild(txt)  // 附加子节点

removeChild()
insertBefore()
replaceChild()

```



给元素对象绑定事件响应：

```javascript
btn.onclick = function(){
  //
}
```

获取元素的html文本：

```javascript
btn.innerHTML
```

HTML元素的属性即元素对象的属性：

```java
btn.value // value属性
btn.name  // name属性
btn.className  // class属性，这个比较特殊，使用的不是html元素的属性名
```

通过元素对象获取子节点：

```javascript
// 节点：包括html标签元素和文本节点
elem.getElementsByTagName('li')  // 子元素
elem.childNodes // 所有子节点的数量，包括空白
elem.firstChild  // 获取第一个子元素,包括空白
elem.lastChild 
elem.children // 获取所有子元素
elem.parentNode  // 父节点
elem.previousSibling  // 前一个兄弟节点
elem.nextSibling
elem.previousElementSibling  // 前一个元素 ie8以上支持

```

修改元素样式：

```javascript
elem.style.width = "300px"
elem.style.backgroundColor = "yellow"  // background-color需要改成驼峰命名，可以参考w3c的css文档
```

需要注意的是，通过上述方式获取样式属性时，只能获取到内联样式，要获取当前实际应用的样式，可以用：

```javascript
var obj = getComputedStyle(elem,null)   // ie8以上支持
console.log(obj.width)
```



### element对象

元素对象表示html的标签元素。

常用属性：

```javascript
clientHeigh // 只读属性，元素的可见高度，包括padding，值为数值类型，不带单位
clientWidth // 只读属性，元素的可见宽度，包括padding

scrollWidth // 滚动宽度
scrollHeigh // 滚动高度
scrollLeft  // 水平滚动条移动的距离
scrollTop   // 垂直滚动条移动的距离

offsetHeigh // 只读属性，元素的高度，包括padding和外边框
offsetWidth // 只读属性，元素的宽度，包括padding和外边框

offsetParent // 定位父元素，返回第一个开启了position属性的父元素，都没有的话就返回body
offsetLeft //相对于定位父元素的水平偏移量
offsetTop //相对于定位父元素的垂直偏移量
```



### event

产生事件时，浏览器都会为事件创建一个对象传给响应函数。通过元素的属性（如btn.onclick）直接绑定响应函数只能绑定一个，绑定多个会被覆盖掉。要绑定多个响应函数，可以用addEventListener("click",func)

常见事件：

```javascript
onmousemove // 鼠标在元素内移动产生的事件
onmousedown
onmouseup
onmousewheel

onkeyup
onkeydown  // 长按会连续触发，第1次触发后会稍微延迟一下，后面触发的速度会非常快
```



event对象属性：

```javascript
clientX  // 事件发生时鼠标的位置
clientY
wheelDelta // 鼠标滚轮滚动的幅度
altKey // alt键是否被按下
shiftKey
ctrlKey

cancelBubble  // true:取消冒泡
```



常用方法：

```javascript
preventDefault()  // 取消默认行为,也可以在绑定响应函数的时候返回false来取消默认行为
```



## BOM对象

操作浏览器的对象。所有bom对象都是全局属性，可以直接使用。并且所有bom对象都是window对象的属性。

### window

代表整个窗口。

常用方法：

```javascript
alert()   
confirm()  // 弹出一个对话框，包含“确认”和“取消”两个选项

var timer = setInterval(callback,1000)  // 每1000毫秒调用一次callback  
clearInterval(timer)  // 清除定时器

setTimeout(callback,2000)  // 延时调用，2s后执行，只会执行一次
```

 ### navigator

代表浏览器本身，可以用来识别不同的浏览器。实际上并不好用。



### location

代表当前地址栏信息，可以获取地址栏信息或者操控浏览器跳转。

```javascript
location = "http://www.baidu.com" // 跳转到百度
location = "inxex.html"   // 相对路径

location.assign()  // 等价于直接修改loaction对象
location.reload(true)  // 重新加载页面，true:强制清空缓存
```





### history

浏览器浏览的历史记录，不能获取到具体的历史记录，只能操作浏览器向前或向后翻页，而且只能访问当次的历史记录。

```javascript
history.length  // 当次访问页面的数量

back() // 回退到上一个页面
forward()  // 跳转到下一个页面
```





### Screen

获取用户显示器相关信息。



## 代码

```javascript
// 创建对象1
obj = new Object()

// 创建对象
obj = {}  // 可以直接指定属性

// 创建一个函数对象
fun = new Function()

// 将函数内容封装在Function对象里
fun = new Function("console.log('helloworld')")
fun() // 执行方法

// 创建匿名函数对象
fun = function(){
  //body
}


// 添加属性
obj.name = 'helloworld'
obj['123'] = 'adf' // 不遵守标识符规范的属性只能用这种方式添加

// 删除对象属性
delete obj.name

// 检查对象是否包含相关属性
"name" in obj


```





# Reference

```javascript
// number类型的常量
Number.MAX_VALUE  // 以及其它常数
Number.MIN_VALUE  // 0以上的最小值
Infinity // 字面量，表示正无穷大
NaN // 表示不是数字

// 基本数据类型的内置方法
toString() // 转成string类型


// 内置方法
String()  // 转成string类型
Number()  // 转成number类型，true:1 false:0 字符串:NaN 空字符串:0 null:0 undefined:NaN
Boolean() // 0和NaN:false 其它数字:true  空字符串:false 其它字符串:true null/undefined:false object:true
parseInt()  // 把整数前缀取出来,可以指定进制
parseFloat()  
isNaN()

// 内建对象
Math
String
...

// DOM/BOM对象
console
document
window

```

