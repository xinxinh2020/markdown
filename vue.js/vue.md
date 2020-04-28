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



## 指令

### v-html

### v-bind

用于设置HTML元素的属性：

```html
<!-- use是个变量 -->
<div v-bind:class="{'class1': use}"></div>
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