

[TOC]

# 基础

## 前世今生

一开始是用RUBY开发的，后来才改用JS



## 基础知识

- 在服务器端，JS可以操作数据库和文件
- REPL：命令行交互界面
- require('./module')可以导入一个模块，参数是模块的文件名(./module.js)
- npm install本地安装的包自动会加到项目的package.json的依赖中去
- 前后端分离前，前端页面写完是要交给后端的（如jsp）
- 原则上，所有包都应该安装到本地，出了少数提供可执行命令并且在多个项目中共同使用的包，如vue-cli，详见：[使用本地还是全局包？](https://nodejs.dev/learn/npm-global-or-local-packages)
- npx用于在项目目录下运行可执行的包命令，详见：[npx](https://nodejs.dev/learn/the-npx-nodejs-package-runner)
- setTimeout会把回调函数放到消息队列（Message Queue）中，消息队列中的函数需要等到调用栈(call stack)中的函数都执行完了以后才会执行，Promise会把回调放在任务队列（Job Queue）中，任务队列中的任务会优先被执行，详见：[event loop](https://nodejs.dev/learn/the-nodejs-event-loop)
- express实例的use方法可以传一个函数作为中间件（类似于拦截器），常用中间件：
  - static：用于设置静态资源目录
  - 用来做路由分发



## 常用命令

```shell
npm config set registry https://registry.npm.taobao.org  # 配置淘宝源
npm config get registry # 验证配置是否成功
npm init --yes # 初始化项目

npm install # 安装package.json中指定的项目依赖到node_modules文件夹下
	--production # 不安装devDependencies
npm install <package-name> # 安装指定包到当前文件树的node_modules目录下，并添加到package.json的dependencies中
	-g # 安装指定包到全局的node_modules中，并且不会自动添加到当前项目的依赖中，项目中不能使用(require)全局包
	--save # 安装指定包并把它添加到package.json的dependencies中，npm5以后无需显式指定该参数
	--save-dev # 安装指定包并把它添加到package.json的devDependencies中
npm root -g # 查看全局node_modules的位置
npm update # 升级项目依赖的包
npm update <package-name> # 升级指定包，升级时package-lock.json会随着更改，而package.json保持不变
npm run <task-name> # 运行package.json的scripts中定义的命令行命令
npm list # 列出已安装的local包,内容和package-lock.json类似
	--depth=0 # 列出依赖的深度，0为不列出依赖
npm list -g # 列出全局安装的包
npm view <package-name> version # 查看指定包的最新版本
npm view <package-name> versions # 查看指定包的所有版本 
npm outdated # 查看过期的包
npm uninstall <package-name> # 删除当前项目下指定包
	-S, --save # 同时删除package.json中的dependencies引用
	-D, --save-dev # 同时删除package.json的devDependencies中
	-g # 删除全局范围的包
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

## 内置模块

### process

参考文档：https://nodejs.org/dist/latest-v12.x/docs/api/process.html



### console

和浏览器的console对象功能类似。参考：https://nodejs.dev/learn/output-to-the-command-line-using-nodejs

- console.log : 打印输出
- console.count：打印输出并统计该输出打印的次数
- console.clear：清屏
- console.trace：打印堆栈
- console.time('doSomething()')/console.timeEnd('doSomething()')：计时
- console.error：错误输出

## 常用第三方库

```shell
nodemon # 使用该工具启动程序后修改代码会自动更新，无需再重启服务
apidoc  # 通过注解自动生成文档
nodemailer # 发送邮件
cheerio # 将HTML字符串转化为类dom,之后可以通过JQuery语法选中其中的元素，可以用于爬虫得到的网页数据解析
mongoose # mongoDB
request # 封装了HTTP请求
typescript # 增强版的javascript，使用强类型编译语法
progress # 进度条
chalk # 字体颜色
readline # 交互式输入
inquirer # 交互式输入

## express相关模块
express # 用于提供服务器端API
body-parser # 用于在express中解析post请求的body 
express-session
multer # 图片上传

## vue相关
vue # import Vue from 'vue'
vue-router
vuex # import Vuex from 'vuex'

## 常用模块
ws # websocket
socket.io # websocket
jsonwebtoken # 生成token
```

## 交互式界面

REPL,Run Evaluate Print Loop

参考文档：https://nodejs.dev/learn/how-to-use-the-nodejs-repl






```

```