[TOC]

# 传参方式

## JavaScript

普通传参：

```javascript
node app.js aaa

// app.js
process.argv.forEach((val, index) => {
    console.log(`${index}: ${val}`)  // 前2个参数分别为node命令所在的路径和app.js所在的路径，第3个参数起为用户传进去的参数
})
```

执行结果：

```shell
D:\workspace\idea\nodejs_demo>node app.js aaa
0: D:\development\nodejs\node.exe
1: D:\workspace\idea\nodejs_demo\app.js
2: aaa
```



键值对传参：

```javascript
node app.js --name=yini

// app.js
// minimist模块用于解析键值对参数
const args = require('minimist')(process.argv.slice(2)) // 去掉前2个固定的参数
console.log(args['name'])
```

 执行结果：

```shell
D:\workspace\idea\nodejs_demo>node app.js --name=yini
yini
```

