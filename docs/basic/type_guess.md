当定义一个变量却不显式指定类型时候，类型由值推导而出

```go
var i int
j := i	//j是int类型
k := 0	//k是int类型
```

但是如果值是数字时候，则新的变量可能是int、float64、complex128，这取决于常量的精度

```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```

`%T`可以查看类型

```go
fmt.Printf("%T", a)
```
