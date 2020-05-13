[TOC]



# 基础

响应式：数据的变化会导致页面内容变化

单向数据：从数据到应用



## html选择器

```javascript
el: '#app'  // id选择器
```

## 数组

用索引的方式改变数组数据不是响应式的，可以通过以下方法来进行响应式的修改：

```javascript
vm.$set(vm.items,index,newValue)
Vue.set(vm.items,index,newValue)
```



### 变异方法（修改原有数据）

- push() : 添加一个元素
- pop() : 删除一个元素
- shift() : 
- unshift()
- splice()：删除指定索引的元素
- sort()
- reverse()

### 替换数组（生成新的数组）

- slice() : 切片，截取数组的部分元素
- concat()
- slice()

## 组件

- 组件的data必须是一个函数

- 组件的模板内容必须是单个根元素
- 模板内容可以使用模板字符串来增加可读性
- 可以使用短横线或者驼峰命名的方式
- props用于父组件通过属性的方式向子组件传值，应该是properities的缩写，父组件向子组件传递数据的方向是单向的
- 子组件可以通过$emit产生一个事件，父组件通过监听该事件来获得子组件传过来的数据 
- 非父子关系的组件通信要经过事件中心（其实就是一个Vue对象）



## 前后端交互

### Promise

- 是一个函数对象
- 在要求异步请求的执行顺序时，可以避免传统方法的回调地狱
- 创建Promise对象时，需要传一个函数（function(resolve, reject )）给它，在函数里执行一些异步操作（如ajax请求），执行成功通过resolve返回结果，失败通过reject返回结果，该对象的then方法可以得到这些结果数据。catch方法可以得到reject数据（then方法也可以，但要写第二个callback来接收，这种方式不推荐）。
- 一组then调用链只要在最后写一个catch就可以了，因为一旦有一个then调用失败，后面的then都不会执行，直接跳到catch中集中处理异常情况。可以在then中手动抛出一个错误（`throw new Error('error!')`）来终止调用链的执行。

常用方法：

- all() : 并发处理多个异步任务，所有任务都完成后才返回
- race() : 并发处理多个异步任务，只要有一个任务完成就返回

### axios

拦截器：

```javascript
// 拦截请求
axios.interceptors.request.use(function(config){
  return config
}, function(err){
  
})

// 拦截响应
axios.interceptors.response.use(function(config){
  return config
}, function(err){
  
})
```



## 路由



# Vue对象



## 属性



```shell
el # 元素的挂载位置，可以是css选择器或者是dom元素
data # 模型数据，值是一个对象
methods # 值是一个对象，里面可以包含很多方法。方法里需要通过this来获取data中的数据，this指向vue对象
computed # 计算属性，值是一个对象，对象里定义各种方法。方法中的数据是基于data中的数据进行计算的，当data中的数据变化时，方法的计算结果也会发生变化。计算属性相比起methods中定义的方法，计算属性是基于缓存的，计算结果会放在缓存里，多次调用计算属性的方法只会进行一次计算（如果data中的数据没有发生变化的话），而普通方法是没有使用缓存的
watch # 侦听器，监听某个属性，当属性变化时，执行一些操作
router # 路由
directives # 局部指令
filters # 局部过滤器，用的时候和linux的管道操作类似
components # 局部组件

## 生命周期
## 挂载
beforeCreate
created
beforeMount
mounted # 比较重要的一个环节，此时模板中的元素都已经加载了，可以往模板里填充数据
## 更新（当data中的数据发生变化时会触发）
beforeUpdate
updated
## 销毁
beforeDestroy
destroyed
```



## 方法

- $set : 响应式地修改数组数据，或者对象的属性
- $on : 监听(自定义)事件
- $off : 销毁事件
- $emit : 触发事件

# 指令

- 数据绑定指令：v-text/v-html/v-model

```html
<div v-text='msg'></div>  <!-- 类似于差值表达式 -->
<div v-html='msg'></div>  <!-- 插入html片段，可能会有安全性问题 -->
<div v-model='msg'></div>  <!-- 双向数据绑定 -->
```

- v-on

绑定事件。

v-on:click也可以简写成@click

将事件对象传给绑定的响应函数：

```html
<button @click='handle'>sumbit</button>  <!-- 直接绑定响应函数名，默认会把event作为第一个参数传给响应函数 -->
<button @click='handle(123,$event)'>sumbit</button>  <!-- 通过这种方式调用响应函数，如果要把事件对象传给响应函数，需要固定在最后一个参数写上$event -->
```



修饰符：

```html
<!-- 点击事件修饰符 -->
<button @click.stop='handle'>sumbit</button>  <!-- 阻止冒泡 -->
<button @click.prevent='handle'>sumbit</button>  <!-- 阻止默认行为 -->

<!-- 键盘事件修饰符 -->
<button v-on:keyup.enter='handle'>sumbit</button>  <!-- 回车键 -->
<button v-on:keyup.delete='handle'>sumbit</button>  <!-- 回车键 -->
<button v-on:keyup.112='handle'>sumbit</button>  <!-- f1键 -->

<!-- 自定义按键修饰符 -->
Vue.config.keyCodes.f1 = 112

<!-- 表单修饰符 -->
<input v-model.number="age" type="number">  <!-- 把值转化为数值类型 -->
<input v-model.trim="info">  <!-- 去掉前后空格 -->
<input v-model.lazy="info">  <!-- 把input事件转化为change事件（失去焦点才会产生事件） -->
```

多个修饰符可以连写

- v-bind

绑定属性：

```html
<!-- 点击事件修饰符 -->
<a v-bind:href='url'>跳转</a>  
<a :href='url'>跳转</a>  <!-- 缩写 -->

<div v-bind:class='{active: isActive}'>跳转</div>  <!-- 绑定样式-对象形式，active是样式名，isActive是data中的数据 -->
<div v-bind:class='[activeClass, errorClass]'>跳转</div>  <!-- 绑定样式-数组形式，activeClass和errorClass是data中的数据，值是对应的样式名 -->
```

- 分支循环
- v-if/v-else-if/v-else

渲染满足条件的元素，不满足的不会渲染

- v-show

不管满不满足条件都会渲染元素，只是满足条件的会显示，不满足的不会显示（style='display:none'）,如果元素需要频繁切换显示/隐藏，用v-show性能会比较好，因为v-if执行的是增加/删除，开销会比较大

- v-for

有以下形式：

```html
<li v-for='item in fruits'>{{ item }}</li>  <!-- 遍历数组 -->
<li v-for='(index,item) in fruits'>{{ index + '---' + item }}</li>  <!-- index是索引 -->
<li v-for='(value, key, index) in obj'>{{ item }}</li>  <!-- 遍历对象 -->
```

v-for可以和v-if结合使用。






- 其他指令

v-once：显示一次数据以后，不再具有数据响应功能，用于不需要修改的数据，可以提高性能



# 特性

## 自定义指令

## 过滤器



