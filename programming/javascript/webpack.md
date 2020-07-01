[TOC]



## 简介

webpack用于打包js应用，它会自动分析应用中各个文件的依赖关系，并把它们打包到一个（或几个）bundle文件中。

## 基础

webpack运行在nodejs上。

依赖图：一个文件依赖另一个文件，webpack会把这当成一个依赖，而不仅仅局限于JS文件

对于webpack来说，项目中的每个文件都是一个模块

webpack处理js应用时，从entry points开始，递归地构建一个依赖图，这个依赖图包含应用需要的所有模块，把这些模块打包到若干个（很多时候只有一个）bundle中，这个bundle会被浏览器加载。

## chunk

https://webpack.js.org/concepts/under-the-hood/#chunks

一个entry point会创建一个main chunk，这个chunk中会包含所有这个entry point依赖的模块

## 模块

几种表达依赖关系的方式：

- ES2015的import语句
- CommonJS的require语句
- AMD的define和require语句
- css/sass/less文件中的@import语句

Loader向webpack描述了如何去处理非Javascript的模块并把这些依赖打包到bundles中。



### Resolver

Resolver用于定位一个模块的绝对路径。webpack使用enhanced-resolve支持解析3种文件路径：

1. 绝对路径：

```shell
import '/home/me/file';

import 'C:\\Users\\me\\file';
```

因为已经是绝对路径，所以实际上已经不需要解析。



2. 相对路径：

```shell
import '../src/file1';
import './file2';
```

以包含上述语句的文件所在的目录作为当前目录

3. 模块路径

```shell
import 'module';
import 'module/lib/file';
```

   搜索所有包含在resolve.modules中定义的目录

找到符合条件的模块后，就会将它们bundle起来。



### Federation





## Why Webpack

一开始，但我们需要使用一个js文件中的函数时，就把这个js文件include进来，但是这样可扩展性很差，导入过多的js文件会导致网络瓶颈。另一个做法是把所有的代码都放到一个js文件中，但这样会导致作用域、可读写和可维护性问题。

IIFE解决了作用域的问题，它把多个文件安全地拼接起来，不用担心作用域冲突的问题。但当你改动一个文件时，就需要重新构建整个项目，同时构建时进行优化也很难，很难知道一段代码使用被使用。

nodejs的出现引入了新的挑战，nodejs应用不运行在浏览器上，没有html文件和脚本标签可以使用。CommonJS引入了require来用于导入模块，解决了作用域问题，浏览器不支持CommonJS。webpack打包JS应用（同时支持CommonJS和ESM）

自动的依赖分析：基于代码中的导入和导出来推断依赖关系图