[TOC]

# 目录

## 基本语法

```javascript
 //文本插值
<div id="app">
	{{ message }}
</div>

//绑定元素特性
<div id="app">
    <span v-bind:title="message">
        this is a span
	</span>
</div>

//条件语句（改变DOM结构）
<div id="app-3">
  <p v-if="seen">现在你看到我了</p>
</div>

//循环语句（绑定数组的数据）
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>

//事件监听
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">逆转消息</button>
</div>

//表单输入和应用状态之间的双向绑定
<div id="app-6">
  <p>{{ message }}</p>
  <input v-model="message">
</div>
```



## Vue对象的属性



### methods

### computed

computed只有相关依赖发生改变时才会重新取值。而使用 methods ，在重新渲染的时候，函数总会重新调用执行。

使用 computed 性能会更好，但是如果你不希望缓存，你可以使用 methods 属性。



## 指令

### v-bind

用于设置HTML元素的属性：

```html
<!-- use是个变量 -->
<div v-bind:class="{'class1': use}"></div>

<!-- 可以缩写 -->
<a :href="url"></a>
```

如下，href 是参数，告知 v-bind 指令将该元素的 href 属性与表达式 url 的值绑定

```html
<div id="app">
    <pre><a v-bind:href="url">菜鸟教程</a></pre>
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    url: 'http://www.runoob.com'
  }
})
</script>
```



#### v-on

用于监听DOM事件：

```html
<a v-on:click="doSomething">
  
<!-- 缩写 -->
<a @click="doSomething"></a>
```

这里的click也是参数，设置指定的监听事件



### v-html

#### v-model

可以实现双向的数据绑定：

```html
<div id="app">
    <p>{{ message }}</p>
    <input v-model="message">
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    message: 'Runoob!'
  }
})
</script>
```



#### v-show

根据条件展示元素：

```html
<h1 v-show="ok">Hello!</h1>
```

类似于v-if的作用。



#### v-for

需要以 **site in sites** 形式的特殊语法， sites 是源数据数组并且 site 是数组元素迭代的别名:

```html
<div id="app">
  <ol>
    <li v-for="site in sites">
      {{ site.name }}
    </li>
  </ol>
</div>
 
<script>
new Vue({
  el: '#app',
  data: {
    sites: [
      { name: 'Runoob' },
      { name: 'Google' },
      { name: 'Taobao' }
    ]
  }
})
</script>
```



## 捕获元素

通过Vue对象的el匹配html的div：

```javascript
	<script type="text/javascript">
		var vm = new Vue({
			el: '#vue_det',
			data: {
				site: "菜鸟教程",
				url: "www.runoob.com",
				alexa: "10000"
			},
			methods: {
				details: function() {
					return  this.site + " - 学的不仅是技术，更是梦想！";
				}
			}
		})
	</script>
```



#### id

```shell
<div id="vue_det">
el: '#vue_det'
```



## 前端基础

### class属性

```css
static active text-danger
```

### style属性

```css
color fontSize display
```

input

```javascript
checkbox //复选框
radio //单选框
```



## SomeThing

new Vue()创建了一个Vue的根实例