[TOC]



# 基础

响应式：数据的变化会导致页面内容变化

单向数据：从数据到应用



## html选择器

```javascript
el: '#app'  // id选择器
```



# Vue对象



## 属性

### el

元素的挂载位置，可以是css选择器或者是dom元素

### data

模型数据，值是一个对象

### methods

值是一个对象，里面可以包含很多方法。方法里需要通过this来获取data中的数据，this指向vue对象。



# 指令

## 数据绑定指令

```html
<div v-text='msg'></div>  <!-- 类似于差值表达式 -->
<div v-html='msg'></div>  <!-- 插入html片段，可能会有安全性问题 -->
<div v-model='msg'></div>  <!-- 双向数据绑定 -->
```

## v-on

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
```

多个修饰符可以连写



### v-bind

绑定属性：

```html
<!-- 点击事件修饰符 -->
<a v-bind:href='url'>跳转</a>  
<a :href='url'>跳转</a>  <!-- 缩写 -->

<div v-bind:class='{active: isActive}'>跳转</div>  <!-- 绑定样式-对象形式，active是样式名，isActive是data中的数据 -->
<div v-bind:class='[activeClass, errorClass]'>跳转</div>  <!-- 绑定样式-数组形式，activeClass和errorClass是data中的数据，值是对应的样式名 -->
```





## 其他指令

v-once：显示一次数据以后，不再具有数据响应功能，用于不需要修改的数据，可以提高性能