有2个很重要的内置函数：`len()`和`cap()`

`len()`可以用来查看数组或slice的长度

`cap()`可以用来查看数组或slice的容量

在数组中由于长度固定不可变，因此`len(arr)`和`cap(arr)`的输出永远相同

在slice中，`len(sli)`表示可见元素有几个（也即直接打印元素看到的元素个数），而`cap(sli)`表示所有元素有几个，比如：

```go
arr := []int{2, 3, 5, 7, 11, 13}
sli := arr[1:4]
fmt.Println(sli)
fmt.Println(len(sli))
fmt.Println(cap(sli))
```

输出

```
[3 5 7]
3
5
```

查看一共有几个元素的方法：

```go
fmt.Println(sli[:cap(sli)])
```

输出

```
[3 5 7 11 13]
```

这样查看有个好处就是对sli没有影响，包括元素内容、len、cap
