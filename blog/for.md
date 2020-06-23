[TOC]

# 各种语言for循环汇总

## Java

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
// 对于map来说，key就是map的键，对于切片和数组来说，key就是下边，这一点go还是很方便的
for key,value := range mySet {
	fmt.Printf("%v : %v\n", key, value)
}
```

