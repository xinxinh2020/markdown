

[TOC]

# 基础

## 前世今生

一开始是用RUBY开发的，后来才改用JS



## 基础知识

- 在服务器端，JS可以操作数据库和文件
- REPL：命令行交互界面
- require('./module')可以导入一个模块，参数是模块的文件名(./module.js)
- module.exports=name，可以把name导出去
- npm install安装的包自动会加到项目的package.json的依赖中去
- 前后端分离前，前端页面写完是要交给后端的（如jsp）
- express实例的use方法可以传一个函数作为中间件（类似于拦截器），常用中间件：
  - static：用于设置静态资源目录
  - 用来做路由分发



## 常用命令

```shell
npm config set registry https://registry.npm.taobao.org  # 配置淘宝源
npm config get registry # 验证配置是否成功
npm init --yes # 初始化项目

# 常用第三方库
npm install nodemon # 使用该工具启动程序后修改代码会自动更新，无需再重启服务
npm install apidoc  # 通过注解自动生成文档
npm install nodemailer # 发送邮件
npm install cheerio # 将HTML字符串转化为类dom,之后可以通过JQuery语法选中其中的元素，可以用于爬虫得到的网页数据解析
npm install mongoose # mongoDB
npm install request # 封装了HTTP请求
npm install typescript # 增强版的javascript，使用强类型编译语法

## express相关模块
npm install express # 用于提供服务器端API
npm install body-parser # 用于在express中解析post请求的body 
npm install multer # 图片上传

## 常用模块
ws # websocket
```



## 常用代码

```javascript
// fs模块
const fs = require('fs') // 引入内置模块
let m = require('./module')  // 引入./module.js中导出的自定义模块
let dirs = fs.readdirSync('./')  // 同步读取目录下的文件，需要try...catch一下，否则出错程序会终止执行
// 异步读取目录下的文件，不需要try...catch,在callback中处理err即可
fs.readdir('./', (err, data) => {
    console.log(err)
    console.log(data)
})
// 创建一个目录
fs.mkdir('./test111', (err) => {
    console.log(err)
})
// 修改文件名
fs.rename('./test111', './test222', (err) => {
    console.log(err)
})
// 删除目录，只能删空目录
fs.rmdir('./test222', (err)=>{
    console.log(err)
})
// 删除文件
fs.unlink('hello.txt', err => {
    console.log(err)
})
// 写入数据到文件中，文件不存在会创建一个文件，文件存在会覆盖原有文件内容
fs.writeFile('hello.txt', 'hi nodejs2', err => {
    console.log(err)
})
// 追加内容到文件中，文件不存在会创建一个
fs.appendFile('hello.txt', 'hi nodejs3', err => {
    console.log(err)
})
// 读取文件内容，返回byte数据
fs.readFile('hello.txt', (err, msg) => {
    console.log(err)
    console.log(msg.toString('utf8'))
})

// url模块
const url = require('url')  // 引入URL模块
let obj = url.parse("https://www.baidu.com/s?ie=utf-8")   // 将URL字符串解析成一个对象
let str = url.format(urlObj) // 将URL对象格式化成字符串

// querystring模块
const qs = require('querystring')
let obj = qs.parse('name=yini&age=20&height=163')  // 将query字符串解析成一个对象
let str = qs.stringify(obj)  // 把对象格式化成字符串
let str = qs.escape('name=你好&age=18&role=学生') // 编码
let str2 = qs.unescape('name%3D%E4%BD%A0%E5%A5%BD%26age%3D18%26role%3D%E5%AD%A6%E7%94%9F')  // 解码
```

