[TOC]

# 各种语言for循环汇总

## Java

普通for循环

```java
int[] arr = {1， 2， 3， 4};
for (int j = 0; j < arr.length; j++) {
    System.out.println(arr[j]);
}
```

foreach

```java
// 遍历集合元素的值
int[] arr = {1， 2， 3， 4};
for (int val : arr) {
    System.out.println(val); // 输出数组的值
}
```



## JavaScript

普通for循环

```javascript
let array = [1, 2, 3, 4];
for(let k = 0; k < array.length; k++) {
   console.log(array[k]);
}
```

foreach

```javascript
let arr = [9, 8, 7, 6]
arr.forEach(function(value, index, array) {
				console.log(value,index,array)
})

arr.forEach(num =>{
    console.log(num)
})
```

for...in

```javascript
// 注意：for...in除了遍历数组元素以外，还会遍历对象的自定义属性
let array = [1, 2, 3, 4];
for(let index in array) {    // index是数组的索引
   console.log(array[index]);
}
```

for...of

```javascript
// ES6新特性，适合遍历数组，Map等集合元素
let array = [1, 2, 3, 4];
for(let value of array) {  // value是数组的值
   console.log(value);
}
```



## Golang

普通for循环

```go
for i:=0; i<1000; i++ {
    fmt.Println(i)
}
```

foreach

```go
//mySet := []int{1,2}
//var mySet [8]int
mySet := map[string]string{"hello":"world"}
// 对于map来说，key就是map的键，对于切片和数组来说，key就是下标
for key,value := range mySet {
	fmt.Printf("%v : %v\n", key, value)
}
```

